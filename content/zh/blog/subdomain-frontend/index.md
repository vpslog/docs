---
title: "基于 Cloudflare API 构建一个次级域名分发系统——前端篇"
description: ""
excerpt: ""
date: 2023-07-04T04:34:24Z
lastmod: 2023-07-04T04:34:24Z
draft: false
weight: 50
images: ["cover.png"]
categories: ["Dev"]
tags: ["Cloudflare","DNS","Domains","Vue","Element-UI"]
contributors: ["Gray"]
pinned: false
homepage: false
---

## Introduction

本次采用 [Element-ui](https://element.eleme.io/#/zh-CN) 构建前端。 Element-ui 是国内 饿了么 公司开源的一款前端组件库，和 Vue2/Vue3 适配，在 Github 上有 50k+ stars。 Element-ui 比较符合国人审美，然后也有很多项目是用这个组件库。例如 [vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/guide/#%E5%8A%9F%E8%83%BD)。

我个人很喜欢 Element-ui 提供的 message 功能，只要 `this.$message.error` 或者 `this.$message.success` 就能打开一个错误/成功提示，自带淡入淡出动画，且可以主动居中排列等。

但总的来说，这次改 Element-ui 没有之前 bootstrap 用的顺手，例如这个 `el-main` 元素不带居中，还是加 css 控制的。

总之各种前端库大同小异，大家看哪个顺眼用哪个。

## Development

前端的元素也比较简单，一个检索栏、一个已注册域名列表，然后加一个点击后出现的对话框来进行设定即可。



```html
<div style="display: flex; align-items: center; text-align: center;  width:80%;margin:0 auto">
    <el-input v-model="prefix" placeholder="前缀"></el-input>
    <el-select v-model="zone" placeholder="顶级域名">
        <el-option
        v-for="item in zones.domains"
        :key="item.name"
        :label="item.name"
        :value="item.name">
        </el-option>
    </el-select>
    <el-button style="margin-left: 10px;" @click="checkDNSRecord(`${prefix}.${zone}`)" type="primary">查询</el-button>
    </div>

    <div>
    <h3>域名列表</h3>
    <el-table  v-if="dnsRecords && dnsRecords.length > 0" :data="dnsRecords">
        <el-table-column prop="name" label="名称"></el-table-column>
        <el-table-column prop="type" label="类型"></el-table-column>
        <el-table-column prop="content" label="内容"></el-table-column>
        <!--参考 https://blog.csdn.net/csdn_jy/article/details/89381393 -->
        <el-table-column prop="proxied" label="是否代理" :formatter="parseProxied"></el-table-column>
        <el-table-column prop="user_id" label="用户ID"></el-table-column>
        <el-table-column>
        <template slot-scope="scope">
            <el-button @click="editDNSRecord(scope.row)" type="text">编辑</el-button>
            <el-button @click="deleteDNSRecord(scope.row)" type="text">删除</el-button>
        </template>
        </el-table-column>
    </el-table>
    <p v-else>这里什么也没有</p>
</div>
```

scope 是 Vue 的一个特性，用于访问局域的变量。在 table 中体现为某一特定行。

弹出对话框可以放在任何地方。默认不展示，通过交互展示，`dialogType`变量用于指定是“创建域名”还是“修改域名”对话框。

```html
<el-dialog :visible.sync="dialogVisible" :title=" dialogType == 'create' ? '创建DNS记录':'修改DNS记录'">
    <el-form :model="DNSRecordForm">
    <el-form-item label="主域名" :label-width="formLabelWidth">
        <el-select v-model="DNSRecordForm.zone" placeholder="请选择" @change="updateFullDomain">
        <el-option
            v-for="item in zones.domains"
            :key="item.name"
            :label="item.name"
            :value="item.name">
        </el-option>
        </el-select>
    </el-form-item>
    <el-form-item label="完整域名" :label-width="formLabelWidth">
        <el-input v-model="DNSRecordForm.name"></el-input>
    </el-form-item>
    <el-form-item label="解析内容" :label-width="formLabelWidth">
        <el-input v-model="DNSRecordForm.content"></el-input>
    </el-form-item>
    <el-form-item label="类型" :label-width="formLabelWidth">
        <el-select v-model="DNSRecordForm.type" placeholder="请选择">
        <el-option
            v-for="item in types"
            :key="item"
            :label="item"
            :value="item">
        </el-option>
        </el-select>
    </el-form-item>
    <el-form-item label="是否代理" :label-width="formLabelWidth">
        <el-switch v-model="DNSRecordForm.proxied"></el-switch>
    </el-form-item>
    
    <el-button v-if="dialogType == 'create' " @click="createDNSRecord(DNSRecordForm)" type="primary">创建</el-button>
    <el-button  v-if="dialogType == 'edit'"  @click="saveDNSRecord(DNSRecordForm)" type="primary">修改</el-button>
    </el-form>
</el-dialog>
```

这里主域名和解析类型用的是选项，其他都用的 Text。没有做表格验证，丢给 Cloudflare 做吧。（可能之后会补一下

JS 方基本没啥好说的，实现了几个接口而已。然后用`this.$message`和`setTimeout`做了一些小的交互优化，使用起来体验更好。例如下面这个查询域名的方法：

```js
checkDNSRecord(dnsRecord){
    axios.post(`/check/${dnsRecord}`)
    .then(response => {
        console.log(response.data);
        if(response.data.available){
        this.$message.success({
            message:'域名可用'
        })
        this.DNSRecordForm.zone = this.zone
        this.DNSRecordForm.name = dnsRecord
        this.dialogType = 'create';
        setTimeout(() => {
            this.dialogVisible = true;
        }, 1000);
        }else {
        this.$message.error({
            message:'域名不可用'
        })
        }
    })
    .catch(error => {
        
        console.error(error);
    });
},
```


## Conclusion

本文实现了次级域名分发系统前端。Element-ui message 方法非常方便，可以考虑使用。