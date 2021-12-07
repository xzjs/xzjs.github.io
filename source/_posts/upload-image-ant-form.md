---
title: 在ant form中上传图片
date: 2021-12-07 09:46:39
tags: react ant-design
categories: 前端
thumbnail: https://gw.alipayobjects.com/zos/rmsportal/KDpgvguMpGfqaHPjicRK.svg
---
## 背景
最近做勇泰商城项目，用了之前没用过的react和ant-design，其余部分大同小异，结果要在表单里上传图片缺犯了难，网上找了一圈也没有找到靠谱的解决方案，遂在这里记录一番

## 动手
#### 样式
先来一个form组件，然后再form的item里加入upload组件
``` html
<Form onFinish={onFinish} form={form}>
    <Form.Item
    label="商品图片"
    name="img"
    valuePropName="img"
    getValueFromEvent={normFile}
    >
    <Upload
        name="file"
        listType="picture-card"
        className="avatar-uploader"
        action="https://upload.qiniup.com"
        showUploadList={false}
        data={getToken}
        onChange={handleChange}
    >
        {form.getFieldValue("img") ? (
        <img className="img"  src={baseURL + form.getFieldValue("img")} alt="" />
        ) : (
        <div>
            <div style={{ marginTop: 8 }}>Upload</div>
        </div>
        )}
    </Upload>
    </Form.Item>
    <Form.Item>
    <Space size="middle">
        <Button type="primary" htmlType="submit">
        提交
        </Button>
        <Button
        onClick={() => {
            form.resetFields();
            setVisible(false);
        }}
        >
        取消
        </Button>
    </Space>
    </Form.Item>
</Form>
```
使用form组件的关联属性form，组件渲染时使用`form.getFieldValue("img")`检测form中是否有相关的值,此时存储的值并不是像往常那样的图片本地地址，而是上传后的图片hash值，若有值，就拼接baserUrl展示图片
```html
<img className="img"  src={baseURL + form.getFieldValue("img")} alt="" />
```
如果没有值的话就显示上传按钮

```html
<div>
    <div style={{ marginTop: 8 }}>Upload</div>
</div>
```
#### 图片上传完后如何将值回填到form表单中
这就需要用到`Form.Item`的属性`getValueFromEvent`,官方文档给的描述是“设置如何将 event 的值转换成字段值”，说实话，看了好几遍，没明白，还好找到了一段示例代码，最后勉强明白，应该是会捕捉内部元素触发的回调，内部的upload组件文件状态改变时会有一个回调，参见[onChange](https://ant.design/components/upload-cn/#onChange)。Form.Item捕捉到这个回调，我们就可以在里面搞些事情拿到上传后的hash值了，如代码中的`getValueFromEvent={normFile}`

```js
const normFile = (info) => {
    if (info.file.status == "done") {
      return info.file.response.key;
    }
  };
```

上述代码种增加了一步`info.file.status=="done"`的判断，因为拿到的事件是文件状态改变的回调，所以只有最后上传完成的一次回调中会带有返回的hash值。

#### 修改数据如何将hash值又转换成图片显示出来
做CURD的都知道，一个表单一般创建和更新共用，刚才解决了创建的问题。其实更新的问题在上文中已经说过了，其实就是使用`form.getFieldValue("img")`，很简单，问题迎刃而解。

## 总结
之所以遇到这个问题其实是对新技术的不熟悉，等熟悉了之后也觉得这个问题没什么了。程序开发人员写的文档往往如此，自己心理跟明镜似的，所以文档写的非常简单，但不熟悉的人来看文档的话，就非常吃力了，要铭记此事，常自省也。
