---
title: "构建一个基于随机用户信息的表单填写工具"
description: ""
excerpt: ""
date: 2023-06-22T05:08:28Z
lastmod: 2023-06-22T05:08:28Z
draft: false
weight: 50
images: ["randomfill.png"]
categories: ["Dev"]
tags: ["Payments"]
contributors: ["Gray"]
pinned: false
homepage: false
---

在一些小网站购买服务的时候，我们往往不希望自己的真实信息泄露。之前我会考虑借助一些网站，但是一个个字段复制起来实在是太不容易了。于是便考虑是否能撰写一个自动填写表单、自动带入随机信息的服务脚本。

## Introduction

要撰写一个浏览器脚本，比较好的方案是直接使用油猴。油猴提供了一个 JS 的环境，可以直接使用常规的语法进行编写，甚至可以调用 jQuery 等外部库。油猴的开发文档也比较丰富。经测试，Chatgpt 也能比较好的生成油猴代码。

随机数据源使用 [randomuser](https://randomuser.me/)。对于任意网页，在加载完成后检测网页中是否存在具有指定字段的 Form，如果存在，则 fix 一个提示在右上角，用户可以点击选择是否需要自动填写信息。

## Development

具体实现如下，当然看这个填充部分确实是有一点啰嗦。主要是需要检测是否存在已经填写的信息。

```js
// ==UserScript==
// @name         自动填充表单（使用随机用户信息）
// @namespace    http://tampermonkey.net/
// @version      1.1
// @description  自动填充表单，并使用随机用户信息进行填写（仅当存在包含地址信息的注册表单时显示确认提示）
// @author       Gray
// @match        http://*/*
// @match        https://*/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    window.addEventListener('load', () => {
        const formCheck = setInterval(() => {
            const addressField = document.querySelector('input[name*="address"]');
            if (addressField) {
                clearInterval(formCheck); // 停止定时器
                autoFillForm(addressField.closest('form')); // 执行自动填充函数
            }
        }, 2000);
    });

    function autoFillForm(form) {
        console.log('found form');
        // 创建包含确认信息的 div 元素
        const confirmDiv = document.createElement('div');
        confirmDiv.style.position = 'fixed';
        confirmDiv.style.top = '10px';
        confirmDiv.style.right = '10px';
        confirmDiv.style.padding = '10px';
        confirmDiv.style.border = '1px solid black';
        confirmDiv.style.backgroundColor = 'white';
        confirmDiv.style.zIndex = '9999';
        confirmDiv.innerHTML = '是否自动填写表单？<br><button id="confirmBtn">确定</button>';

        // 将 div 元素添加到页面中
        document.body.appendChild(confirmDiv);

        // 添加事件监听器来处理按钮点击事件
        const confirmBtn = document.getElementById('confirmBtn');
        confirmBtn.addEventListener('click', () => {
            // 从 API 获取随机用户信息并填充表单
            fetch('https://randomuser.me/api?nat=us')
                .then(response => response.json()) // 将响应转换为 JSON 格式
                .then(data => {
                    const user = data.results[0]; // 提取第一个结果

                    // 填充表单
                    fillForm(form, user);

                    // 删除确认提示 div 元素
                    document.body.removeChild(confirmDiv);
                })
                .catch(error => console.log(error));
        });
    }

    function fillForm(form, user) {
        const firstNameInput = form.querySelector('input[name="firstname"]');
        if (firstNameInput && !firstNameInput.value) {
            firstNameInput.value = user.name.first;
        }

        const lastNameInput = form.querySelector('input[name="lastname"]');
        if (lastNameInput && !lastNameInput.value) {
            lastNameInput.value = user.name.last;
        }

        const emailInput = form.querySelector('input[name="email"]');
        if (emailInput && !emailInput.value) {
            emailInput.value = user.email;
        }

        const phoneNumberInput = form.querySelector('input[name="phonenumber"]');
        if (phoneNumberInput && !phoneNumberInput.value) {
            phoneNumberInput.value = user.phone;
        }

        const addressInput = form.querySelector('input[name*="address"]');
        if (addressInput && !addressInput.value) {
            addressInput.value = user.location.street.name;
        }

        const address2Input = form.querySelector('input[name="address2"]');
        if (address2Input && !address2Input.value) {
            address2Input.value = user.location.street.number;
        }

        const cityInput = form.querySelector('input[name="city"]');
        if (cityInput && !cityInput.value) {
            cityInput.value = user.location.city;
        }

        const stateInput = form.querySelector('input[name="state"]');
        if (stateInput && !stateInput.value) {
            stateInput.value = user.location.state;
        }

        const postcodeInput = form.querySelector('input[name="postcode"]');
        if (postcodeInput && !postcodeInput.value) {
            postcodeInput.value = user.location.postcode;
        }

        const countryInput = form.querySelector('input[name="country"]');
        if (countryInput && !countryInput.value) {
            countryInput.value = user.location.country;
        }
    }
})();
```


## Discussion

以上脚本会随机生成邮件地址，需要在填充完毕后手动修正下地址。否则会绑定错误的邮件。我之前因为这个原因丢了一个账号。其实比较好的方法是加一个 Catch-all 的域名邮箱，对每一个服务使用不同的邮件地址。例如从 `windows.location.href` 获取网站名，再手动指定一个 `EMAIL_DOMAIN` 变量，组合成可用的邮件地址。

randomuser 这个接口支持自定义国家范围。一种想法是，利用此接口，结合 [GeoIP](https://github.com/P3TERX/GeoLite.mmdb) 数据接口/数据库，能够实现生成符合当前 IP 地址的用户信息，并由此绕过 MaxMind 检查，实现真正的随机信息通行和隐私保护。当然可能要结合一些 POSTID 的生成，或许可以利用 Google Map 的 API。

## Conclusion

本脚本实现了表单的自动填写功能。这个脚本主要是我自己买 VPS 比较多所以顺手弄了一个。其实适用范围也不是很广，存在误报/漏报的情况。欢迎进一步修复。