---
title: 逆向：记录爬取CNVD过程
date: 2025-05-30 16:02:04
categories:
tags:
---

CNVD 网站具有很强大的反爬机制，它的检测流程如下：

1. 发起请求，第一次返回 script 将设置 cookie，并重定向 pathname 和 search

```html
<script>
    document.cookie = ('_') + ('_') + ('j') + ('s') + ('l') + ('_') + ('c') + ('l') + ('e') + ('a') + ('r') + ('a') + ('n') + ('c') + ('e') + ('_') + ('s') + ('=') + (-~false + '') + (9 - 1 * 2 + '') + ((2) * [2] + '') + (-~[7] + '') + (([2] + 0 >> 2) + '') + (9 + '') + ((1 << 1) + '') + (2 + '') + (1 + 2 + '') + (-~[5] + '') + ('.') + (+!+[] * 2 + '') + ((1 + [2] >> 2) + '') + ((1 | 2) + '') + ('|') + ('-') + (-~false + '') + ('|') + ('k') + ('n') + ('I') + ('E') + ('A') + ('v') + ('t') + ('Y') + ('W') + ('g') + ('W') + ('B') + ('b') + ('t') + ('R') + ('t') + ('l') + ('N') + ('c') + ('y') + ('Q') + ('P') + ([2] * (3) + '') + ('n') + ('X') + ((1 | 2) + '') + ((2) * [2] + '') + ('%') + ((2 ^ 1) + '') + ('D') + (';') + (' ') + ('M') + ('a') + ('x') + ('-') + ('a') + ('g') + ('e') + ('=') + (-~[2] + '') + ((1 + [2]) / [2] + '') + (~~[] + '') + (~~'' + '') + (';') + (' ') + ('P') + ('a') + ('t') + ('h') + ('=') + ('/') + (';') + (' ') + ('S') + ('a') + ('m') + ('e') + ('S') + ('i') + ('t') + ('e') + ('=') + ('N') + ('o') + ('n') + ('e') + (';') + (' ') + ('S') + ('e') + ('c') + ('u') + ('r') + ('e');
    location.href = location.pathname + location.search
</script>
```

2. 利用浏览器的二次请求，携带 cookie 从服务器获取反爬脚本（cookie 加密并选取时间因子生成【动态更新】、**环境监测**），利用脚本生成 cookie，此时的请求才会接入正常的网站服务

只要脚本足够的像人类行为，脚本就检测不出来。为了绕开检测，使用 playwright 应该设置浏览器启动参数：

```ts
browser = await chromium.launch({
  slowMo: 100, // 添加延迟模拟人类操作,单位毫秒
  args: ["--disable-blink-features=AutomationControlled"],
});

const context = await browser.newContext({
  userAgent:
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
  javaScriptEnabled: true,
});
await context.addInitScript(() => {
  Object.defineProperty(navigator, "webdriver", { get: () => false });
});
```


### 示例代码
client.ts
```ts
import { genOptionMap } from "@rtpackx/core";
import { TaskPolicy } from "src/enums";

export interface ClientOption {
  username: string;
  password: string;
  base_url?: string;
}

export default abstract class Client {
  constructor(option: ClientOption) {}

  public abstract start(mode: TaskPolicy): void;
  public abstract stop(): void;
}
```

cnvd.ts
```ts
import axios, { type AxiosInstance } from "axios";
import {
  chromium,
  Browser,
  BrowserContext,
  LaunchOptions,
  Page as BrowserPage,
} from "playwright";
import sharp from "sharp";
import * as fs from "fs";
import Path, { resolve } from "path";
import { resolve as urlResolve } from "url";
import { Unit } from "@rtpackx/core";
import { logger } from "src/utils/logger";
import OCR from "src/utils/ocr";
import {
  getBase64Content,
  HttpResponse,
  ensureDirSync,
  FileLinkOption,
  ensureFileSync,
  FileLinkOptionMap,
  writeFileSyncWithLock,
  getRootPath,
} from "src/utils";
import { TaskPolicy } from "src/enums";
import Client, { ClientOption } from "./client";
import Spider, { SpiderOption, SpiderTask } from "./spider";

const ROOT = getRootPath();
const CACHE_PATH = resolve(ROOT, `.cache`);
const CNVD_CACHE_PATH = resolve(CACHE_PATH, `cnvd`);
const CNVD_CACHE_XML_DOWNLOAD_PATH = resolve(CNVD_CACHE_PATH, `download`);

export interface CNVDClientOption extends ClientOption {
  chromiumPath?: string;
  filePolicy?: TaskPolicy;
  retryPolicy?: TaskPolicy;
}

export class CNVDClient implements Client {
  public base_url = "https://www.cnvd.org.cn";
  private session: AxiosInstance;
  private browser: Browser;
  private context: BrowserContext;
  private page: BrowserPage;

  constructor(
    private config: CNVDClientOption,
    private options: LaunchOptions = { headless: false }
  ) {
    this.config.filePolicy ||= TaskPolicy.CACHE_FIRST;
    this.config.retryPolicy ||= TaskPolicy.LIMITED_RETRY;
    this.base_url = this.config.base_url || this.base_url;
  }

  private async humanlike(
    type:
      | "scroll-bottom"
      | "scroll-top"
      | "click-body"
      | "timeout"
      | "reload"
      | "go-home"
      | "go-back" = "timeout",
    option?: { timeout?: number }
  ) {
    switch (type) {
      case "scroll-bottom":
        await this.page.evaluate(() => {
          window.scrollTo({
            left: 0,
            top: document.body.scrollHeight,
            behavior: "smooth",
          });
        });
        break;
      case "scroll-top":
        await this.page.evaluate(() => {
          window.scrollTo({ left: 0, top: 0, behavior: "smooth" });
        });
        break;
      case "click-body":
        await this.page.click("body");
        break;
      case "timeout":
        await this.page.waitForTimeout(option?.timeout || 3000);
        break;
      case "reload":
        await this.page.reload({ waitUntil: "networkidle" });
        break;
      case "go-home":
        await this.page.goto(this.base_url, { waitUntil: "networkidle" });
        break;
      case "go-back":
        await this.page.goBack({ waitUntil: "networkidle" });
        break;
    }
  }

  private async getExitBtn() {
    const exitSelector = `body > div.wrap > div.mw.newheader > div > a:nth-child(3)`;
    const exitBtn = await this.page.$(exitSelector);
    const exitBtnText = await exitBtn?.innerText();
    return exitBtnText?.includes("退出") ? exitBtn : null;
  }

  public async login() {
    const url = `${this.base_url}/user/login`;

    const MAX_RETRY = 10;
    for (let i = 0; i < MAX_RETRY; i++) {
      try {
        await this.humanlike("timeout", { timeout: 6000 });
        const result = await this.loginSubmitForm(url);
        if (result) {
          logger.info("登录成功");
          return true;
        }

        if (!result) {
          logger.error("登录失败");
        }
      } catch (error) {
        logger.error(String(error));
      } finally {
        await this.page.reload({ waitUntil: "networkidle" });
      }
    }

    return false;
  }
  private async loginSubmitForm(url: string): Promise<boolean> {
    const FORM_ID = "#loginForm";
    const EMAIL_ID = "#email";
    const PASSWORD_ID = "#password";
    const VERIFY_CODE_ID = "#codeSpan";
    const CODE_INPUT_ID = "#myCode";
    const codeSrcRegex = `**/common/myCodeNew`;
    this.page.goto(url); // 不能使用 await，避免 waitForResponse 不生效

    // ocr 验证码
    const codeResp = await this.page.waitForResponse(codeSrcRegex);
    await this.page.waitForLoadState("networkidle");
    await this.page.waitForTimeout(2000);
    await this.page.waitForSelector(VERIFY_CODE_ID);
    const buffer = await codeResp.body();
    const base64Data = getBase64Content(buffer);
    sharp(buffer).toFile("cnvd-latest-code.png");
    const resp = await this.getVerificationCode(base64Data);
    if (!resp || (resp && resp.success === false)) {
      return;
    }
    await this.page.fill(EMAIL_ID, this.config.username, { force: true });
    await this.humanlike("timeout", { timeout: 2000 });
    await this.page.fill(PASSWORD_ID, this.config.password, { force: true });
    await this.humanlike("timeout", { timeout: 2000 });
    await this.page.fill(CODE_INPUT_ID, resp.data, { force: true });
    await this.humanlike("timeout", { timeout: 2000 });
    this.page.click("#loginForm > div > p.btn_wrap > a");

    // 根据接口判断是否登录成功
    const userinfoUrl = `${this.base_url}/user/getLoginInfo`;
    const loginResp = await this.page.waitForResponse(userinfoUrl);
    return (await loginResp.json())?.code === "200";
  }
  private async getVerificationCode(image: string) {
    try {
      const res = await new OCR({ mode: "ddddocr" }).recognize(image, "base64");
      if (res.success) logger.info(`获取验证码成功：${JSON.stringify(res)}`);
      else logger.error(`获取验证码失败：${JSON.stringify(res)}`);
      return res;
    } catch (error) {
      logger.error(`获取验证码失败：${String(error)}`);
    }
    return null;
  }

  public async retryLogin(delay = 60000) {
    await this.humanlike("go-home");
    await this.page.reload({ waitUntil: "networkidle" });
    await this.humanlike("timeout", { timeout: 3000 });
    if (this.getExitBtn()) {
      return;
    }
    await this.humanlike("timeout", { timeout: delay });
    await this.login();
  }

  public async getXmlFiles() {
    if (this.config.filePolicy === TaskPolicy.CACHE_ONLY) {
      return;
    }

    const url = `${this.base_url}/shareData/list`;
    await this.page.goto(url, { waitUntil: "networkidle" });
    await this.page.waitForTimeout(3000);
    const total = Number(
      (await (await this.page.$("#patchList > .pages")).innerText()).match(
        /共.*?(\d+).*?条[^>]*/
      )[1]
    );

    const PAGE_SIZE = 100;
    let pageRetryCount = 0;
    let links = [] as FileLinkOption[];

    for (let page = 0; page < total / PAGE_SIZE; page++) {
      await this.humanlike("timeout", { timeout: 6000 });
      if (pageRetryCount++ > Math.max(total / PAGE_SIZE + 10, 20)) {
        logger.error(`获取下载链接失败，重试次数过多`);
        return;
      }

      try {
        const res = await this.parseXmlLinks(url, {
          max: PAGE_SIZE,
          offset: page * PAGE_SIZE,
        });
        links.push(...res);
        logger.info(`获取第 ${page + 1} 页 ${res.length} 个下载链接`);
        await this.humanlike("timeout", { timeout: 3000 });
        await this.humanlike("scroll-bottom");
        await this.humanlike("timeout", { timeout: 3000 });
        await this.humanlike("click-body");
      } catch (error) {
        logger.error(error);
        page--; // 重新获取
        await this.humanlike("timeout", { timeout: 10000 });
        await this.retryLogin();
      }
    }
    logger.info(`获取到 ${total} 个下载链接`);

    await this.scanRepetitiveLinks(links);

    const CACHE_JSON_PATH = resolve(CNVD_CACHE_PATH, "cache.json");
    let cachejson = await this.getCacheJson(CACHE_JSON_PATH);

    if (this.config.filePolicy === TaskPolicy.CACHE_FIRST) {
      cachejson = await this.initCacheJson(CACHE_JSON_PATH, links);
      // 过滤掉已经下载过的文件
      links = links.filter((link) => {
        const exist = cachejson[link.name];
        if (exist) logger.info(`${link.name} 已存在，跳过下载`);
        return !exist;
      });
    }

    let startTime = Date.now();
    let downloadRetryCount = 0;
    for (let i = 0; i < links.length; i++) {
      await this.humanlike("timeout", { timeout: 6000 });
      if (downloadRetryCount++ > Math.max(links.length + 10, 20)) {
        logger.error(`下载文件失败，重试次数过多`);
        return;
      }

      if (Date.now() - startTime > 1000 * 60 * 2) {
        startTime = Date.now();
        await this.humanlike("go-home");
        await this.humanlike("reload");
        await this.humanlike("timeout", { timeout: 3000 });
        await this.page.goto(url, { waitUntil: "networkidle" });
        await this.humanlike("timeout", { timeout: 3000 });
      }

      const link = links[i];
      try {
        const url = urlResolve(this.base_url, link.url);
        await this.downloadXml(url);
        cachejson[link.name] = { url, name: link.name };
        writeFileSyncWithLock(
          CACHE_JSON_PATH,
          JSON.stringify(cachejson, null, 0)
        );
        logger.info(
          `${link.name} 下载完成，${total - links.length + i + 1}/${total}`
        );
        await this.humanlike("click-body");
        await this.humanlike("timeout", { timeout: 3000 });
      } catch (error) {
        logger.error(error);
        i--; // 重新下载
        await this.humanlike("timeout", { timeout: 10000 });
        await this.retryLogin();
      }
    }
    logger.info(`已缓存所有文件`);
  }

  private async parseXmlLinks(
    _url: string,
    params: { max: number; offset: number }
  ) {
    const urlIns = new URL(_url);
    urlIns.searchParams.append("max", params.max.toString());
    urlIns.searchParams.append("offset", params.offset.toString());
    const url = urlIns.toString();

    this.page.goto(url);
    const listResp = await this.page.waitForResponse(
      `${this.base_url}/shareData/list**`
    );

    const pattern =
      /<a\s+href="(\/shareData\/download\/.*?[^"]+)"[^>]*>([^<]+)<\/a>/g;
    const result = [] as FileLinkOption[];
    const html = await listResp.text();
    for (const match of html.matchAll(pattern)) {
      result.push({ name: match[2], url: match[1] });
    }

    return result;
  }

  private async scanRepetitiveLinks(links: FileLinkOption[]) {
    const countIns = { map: {}, repeatSet: new Set<string>() };
    for (const link of links) {
      if (countIns.map[link.name]) {
        countIns.repeatSet.add(link.name);
        countIns.map[link.name] = countIns.map[link.name] + 1;
      } else {
        countIns.map[link.name] = 1;
      }
    }

    for (const name of countIns.repeatSet) {
      logger.info(`${name} 文件重复：${countIns.map[name]}`);
    }
  }

  private async downloadXml(url: string) {
    const filenameReg = /^attachment;filename="(.+)"$/;
    const resp = await this.page.request.get(`${url}`);
    const disposition = resp.headers()["content-disposition"];
    const fileName = decodeURI(
      filenameReg.test(disposition)
        ? filenameReg.exec(disposition)![1]
        : disposition.substring(
            disposition.indexOf("filename=") + 9,
            disposition.length
          )
    );
    const path = resolve(CNVD_CACHE_XML_DOWNLOAD_PATH, fileName);
    const buffer = await resp.body();
    writeFileSyncWithLock(path, buffer);
  }

  private async getCacheJson(path: string) {
    ensureFileSync(path);
    const cachejson = JSON.parse(
      fs.readFileSync(path, "utf-8") || "{}"
    ) as FileLinkOptionMap;

    return cachejson;
  }

  private async initCacheJson(json_path: string, links: FileLinkOption[]) {
    // 已存在的文件生成 cachejson
    const files = fs
      .readdirSync(CNVD_CACHE_XML_DOWNLOAD_PATH)
      .filter((file) => Path.extname(file) === ".xml");

    const cachejson = {} as FileLinkOptionMap;

    const fileSet = new Set(files);
    for (const link of links) {
      if (fileSet.has(link.name)) {
        cachejson[link.name] = {
          url: urlResolve(this.base_url, link.url),
          name: link.name,
        };
      }
    }
    writeFileSyncWithLock(json_path, JSON.stringify(cachejson, null, 0));
    logger.info(`已生成缓存文件 ${json_path}`);
    return cachejson;
  }

  async start(mode?: TaskPolicy) {
    ensureDirSync(CNVD_CACHE_PATH);

    this.browser = await chromium.launch({
      slowMo: 100, // 添加延迟模拟人类操作,单位毫秒
      args: ["--disable-blink-features=AutomationControlled"], // 禁用自动化控制特征
      ...this.options,
    });
    this.context = await this.browser.newContext({
      userAgent:
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
      javaScriptEnabled: true,
    });
    // await this.context 设置下载行为
    this.page = await this.context.newPage();
    await this.context.addInitScript(() => {
      Object.defineProperty(navigator, "webdriver", { get: () => false });
    });

    // 初始化网站脚本执行
    await this.page.goto(this.base_url, {
      waitUntil: "networkidle",
      timeout: 60000,
    });
    // 等待 cnvd.org.cn 初始脚本执行完成
    await this.page.waitForTimeout(6000);
    const exitBtn = await this.getExitBtn();
    if (exitBtn) {
      logger.info("已登陆CNVD网站");
      return void 0;
    }

    await this.login();
    await this.page.waitForTimeout(2000);
    await this.page.hover("#mega-menu-9 > li:nth-child(6) > a"); // humanlike
    await this.page.waitForTimeout(1000);
    await this.page.click(
      "#mega-menu-9 > li:nth-child(6) > div > ul > li:nth-child(3) > a"
    );
    await this.getXmlFiles();
    await this.stop();
  }
  async stop() {
    await this.page.close();
    await this.context?.close();
    await this.browser?.close();
  }
}

export class CNVDSpiderParseVulnIDTask implements SpiderTask {
  constructor(
    private id: string,
    private cnvd_id: string,
    private context: BrowserContext,
    private retryCount?: number
  ) {}

  public async run() {}
}
export class CNVDSpiderParseVulnInfoTask implements SpiderTask {
  constructor(
    private id: string,
    private cnvd_id: string,
    private context: BrowserContext,
    private retryCount?: number
  ) {}

  public async run() {}
}

export interface CNVDSpiderOption extends SpiderOption {
  contextNum?: number;
}
export class CNVDSpider implements Spider {
  public base_url = "https://www.cnvd.org.cn";
  private browser: Browser;
  private contexts: BrowserContext[] = [];
  private taskQueue: SpiderTask[] = [];
  private taskProcessing = false;

  constructor(
    private config: CNVDSpiderOption = {},
    private options: LaunchOptions = { headless: false }
  ) {
    this.base_url = this.config.base_url || this.base_url;
    this.config.contextNum ||= 20;
  }
  public async start() {
    this.browser = await chromium.launch({
      slowMo: 100, // 添加延迟模拟人类操作,单位毫秒
      args: ["--disable-blink-features=AutomationControlled"], // 禁用自动化控制特征
      ...this.options,
    });

    // 生成 context 实例池
    await this.initContexts();
  }

  private async initContexts() {
    const handlers = [] as Promise<BrowserContext>[];
    for (let i = 0; i < this.config.contextNum; i++) {
      handlers.push(this.browser.newContext());
    }
    this.contexts = await Promise.all(handlers);
  }

  private enqueue(task: SpiderTask) {
    this.taskQueue.push(task);
    if (!this.taskProcessing) {
      this.processQueue();
    }
  }

  private async processQueue(): Promise<void> {
    this.taskProcessing = true;
    while (this.taskQueue.length > 0) {
      const task = this.taskQueue.shift();

      if (task) {
        const context = this.contexts.shift();

        // 移除 contextIndex，通过 pop push 来实现循环队列，这样的好处是方便创建/销毁 context，需要基础 context.close()

        try {
          await this.executeTask(task, context);
        } catch (error) {
          console.error(`Task ${task.id} failed:`, error);
          if (
            task.retryCount === undefined ||
            task.retryCount < this.maxRetries
          ) {
            // 增加重试次数并重新入队
            const newTask = { ...task, retryCount: (task.retryCount || 0) + 1 };
            console.log(
              `Retrying task ${task.id}, attempt ${newTask.retryCount}`
            );
            this.enqueue(newTask);
          } else {
            console.error(
              `Task ${task.id} exceeded max retries (${this.maxRetries})`
            );
          }
        }
      }
    }
    this.taskProcessing = false;
  }

  private async getAllCNVD() {
    return 0;
  }
  public async stop() {}
}
```