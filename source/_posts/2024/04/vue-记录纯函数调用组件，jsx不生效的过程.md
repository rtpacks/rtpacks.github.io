---
title: 'vue: 记录纯函数调用组件，jsx不生效的过程'
date: 2024-04-12 23:31:06
categories: Vue
tags:
- jsx
---

今天遇到以函数调用形式调用 `tsx` 组件，没想到编译后的代码不能获取到对应的组件。这是因为 `resolveComponent` 只解析已在 Vue 实例中全局注册的组件。在 `template` 或 `script setup` 中能正常解析 `tsx` 组件是因为在 vite 中配置了对应的解析插件。

经过这个案例，加深了对 Vue 编译过程的了解。

**调用方式**

```typescript
const onDeleteRole = async (id: number) => {
    validatePasswd("是否确认删除？", async (passwd) => {
        await deleteRole(id, rsaEncrypt(passwd) as string);
        listRef.value?.reload();
        Message.success("操作成功");
    });
};
```



**错误代码**

```tsx
import { ref, h } from "vue";
import useLoading from "@/hooks/loading";
import { Modal, FormInstance } from "@arco-design/web-vue";

const validatePasswd = (
  title: string,
  callback: (passwd: string) => Promise<unknown> | unknown,
  validate?: (passwd: string) => Promise<unknown> | unknown,
) => {
  const { loading, setLoading } = useLoading();
  const form = ref({ passwd: "" });
  const formRef = ref<FormInstance>();
  console.log(<a-spin></a-spin>); // undefined
  Modal.confirm({
    title,
    hideCancel: false,
    width: 440,
    modalClass: "passwd-modal",
    content: () => (
      <a-spin loading={loading.value}>
        {console.log(h("a-form"))}
        <a-form ref={formRef} model={form.value} auto-label-width>
          <a-form-item
            label="校验密码"
            required
            rules={{
              required: true,
              validator: (value: string, cb) => {
                if (value) {
                  cb();
                } else {
                  cb("当前用户密码不能为空");
                }
              },
            }}
          >
            <a-input-password
              v-model={[form.value.passwd, "modelValue"]}
              placeholder="请输入当前用户密码"
            ></a-input-password>
          </a-form-item>
        </a-form>
      </a-spin>
    ),
    onBeforeOk: async () => {
      const error = await formRef.value?.validate();
      if (error) return false;
      try {
        setLoading(true);
        const passport1 = await validate?.(form.value.passwd);
        if (passport1 === false) return false;
        const passport2 = await callback(form.value.passwd);
        if (passport2 === false) return false;
        return true;
      } catch (error) {
        console.error(error);
        return false;
      } finally {
        setLoading(false);
      }
    },
  });
};

export default validatePasswd;
```



**正确代码**

```tsx
import { ref, h } from "vue";
import useLoading from "@/hooks/loading";
import {
  Modal,
  FormInstance,
  Spin,
  Form,
  FormItem,
  InputPassword,
} from "@arco-design/web-vue";

const validatePasswd = (
  title: string,
  callback: (passwd: string) => Promise<unknown> | unknown,
  validate?: (passwd: string) => Promise<unknown> | unknown,
) => {
  const { loading, setLoading } = useLoading();
  const form = ref({ passwd: "" });
  const formRef = ref<FormInstance>();
  Modal.confirm({
    title,
    hideCancel: false,
    width: 440,
    modalClass: "passwd-modal",
    content: () => (
      <Spin loading={loading.value}>
        <Form ref={formRef} model={form.value} auto-label-width>
          <FormItem
            label="校验密码"
            required
            field="passwd"
            rules={{
              required: true,
              validator: (value: string, cb) => {
                if (value) {
                  cb();
                } else {
                  cb("当前用户密码不能为空");
                }
              },
            }}
          >
            <InputPassword
              v-model={[form.value.passwd, "modelValue"]}
              placeholder="请输入当前用户密码"
            ></InputPassword>
          </FormItem>
        </Form>
      </Spin>
    ),
    onBeforeOk: async () => {
      const error = await formRef.value?.validate();
      if (error) return false;
      try {
        setLoading(true);
        const passport1 = await validate?.(form.value.passwd);
        if (passport1 === false) return false;
        const passport2 = await callback(form.value.passwd);
        if (passport2 === false) return false;
        return true;
      } catch (error) {
        console.error(error);
        return false;
      } finally {
        setLoading(false);
      }
    },
  });
};

export default validatePasswd;
```

