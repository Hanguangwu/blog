---
title: MERN全栈项目——TravelStory简述
description: 本文简要描述了MERN全栈项目——TravelStory相关教程。
date: 2024-09-28T18:34:25+08:00
draft: false
categories:
- 全栈开发
- React
---

## 前言

原作者用的是commonJS语法， 代码结构也很混乱特别是后端部分，这里按照我在别处学到的内容做了一些修改，确保一个文件中不会有太多代码。

作者没有开源源代码，这里的代码可能存在一些小问题。

主要技术栈：MongoDB, Express, React, Node.js

## 简介

[Full Stack Travel Story App Using MERN Stack | MongoDB, Express, React, Node.js | MERN Project——YouTube视频链接](https://www.youtube.com/watch?v=hI35ICP6ioc)

{{< youtube hI35ICP6ioc >}}

In this video, we will build a Full Stack Travel Story App using the MERN stack (MongoDB, Express, React, Node.js). This app allows users to sign up, log in, and create personal travel stories with features like image uploads, and adding travel date. We also implement search functionality to find stories, filter by date range, and pin favorite stories to the top. Users can also edit or delete their stories.. 

The backend features secure JWT authentication, MongoDB for storing user data and travel stories, and APIs for adding, editing, deleting stories, and uploading images. This tutorial covers the entire development process, from setting up the frontend and backend to integrating APIs and building the UI components.

## 使用教程

git clone 整个项目

添加.env配置信息

cd backend 先执行`npm install`再执行`npm start`

cd frontend 先执行`npm install`再执行`npm run dev`

默认打开`http://localhost:5173`

# TimeStamps

## Frontend React Project Setup

### 00:00 - Demo of Travel Story App

### 04:50 - Frontend React app setup

执行如下指令：

```
mkdir TravelStory
cd TravelStory
mkdir frontend
mkdir backend
cd frontend
```

执行`npm create vite@latest .`创建项目，选择React和JavaScript。

然后执行如下指令：

```
npm install
npm run dev
```

输出结果如下所示：

```
added 264 packages in 3m

102 packages are looking for funding
  run `npm fund` for details
```

在src目录下创建components、pages和utils目录，在pages目录下创建Home、Login和SignUp页面

安装VSCode插件`ES7+ React/Redux/React-Native snippets`

新建.jsx文件使用rafce快速生成相应内容

删除src目录下的App.css文件

修改index.css文件和App.jsx文件

添加google font Poppins

https://fonts.google.com/specimen/Poppins

在index.css文件开头加入如下内容：

```css
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
```

### 08:41 - Tailwind CSS setup

vite 版本 

[Install Tailwind CSS with Vite](https://tailwindcss.com/docs/guides/vite)

Install tailwindcss and its peer dependencies, then generate your tailwind.config.js and postcss.config.js files.

```
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

即在frontend目录下执行`npm install -D tailwindcss postcss autoprefixer`

再执行`npx tailwindcss init -p`

输出结果如下所示：

```
added 85 packages in 7s

125 packages are looking for funding
  run `npm fund` for details

Created Tailwind CSS config file: tailwind.config.js
Created PostCSS config file: postcss.config.js
```

Configure your template paths

Add the paths to all of your template files in your tailwind.config.js file.

tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Add the Tailwind directives to your CSS

Add the @tailwind directives for each of Tailwind’s layers to your ./src/index.css file.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

此时`index.css`文件内容如下所示：

```css
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: "Poppins", "sans-serif";
  }

  body {
    background-color: #fdfeff;
    overflow-x: hidden;
  }
}
```

此时`tailwind.config.js`内容如下所示：

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    fontFamily: {
      display: ["Poppins", "sans-serif"],
    },
    extend: {
      // Colors used in the project
      colors: {
        primary: "#05B6D3",
        secondary: "#EF863E",
      }
    },
  },
  plugins: [],
}
```

### 11:33 - react-router-dom installation & setup

执行`npm i react-route-dom`


## Backend

### 14:32 - Backend Node.js Express project setup

在`backend`目录下执行`npm init -y`

输出信息如下所示：

```
Wrote to /home/xxxxxxxx/TravelStory/backend/package.json:

{
  "name": "travelstory",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": ""
}
```

在`backend`目录下执行`npm i express mongoose jsonwebtoken dotenv cors nodemon bcrypt`

输出结果如下所示：

```
npm warn deprecated inflight@1.0.6: This module is not supported, and leaks memory. Do not use it. Check out lru-cache if you want a good and tested way to coalesce async requests by a key value, which is much more comprehensive and powerful.
npm warn deprecated npmlog@5.0.1: This package is no longer supported.
npm warn deprecated rimraf@3.0.2: Rimraf versions prior to v4 are no longer supported
npm warn deprecated are-we-there-yet@2.0.0: This package is no longer supported.
npm warn deprecated glob@7.2.3: Glob versions prior to v9 are no longer supported
npm warn deprecated gauge@3.0.2: This package is no longer supported.

added 183 packages in 9s

22 packages are looking for funding
  run `npm fund` for details
```

在`backend`目录下创建`index.js`

修改`package.json`文件，增加如下内容：`"start": "nodemon index.js"`

创建`models`文件夹和`.env`文件

此时`index.js`文件内容如下所示：

```js
const bcrypt = require("bcrypt");
const express = require("express");
const cors = require("cors");

const jwt = require("jsonwebtoken");

const app = express();
app.use(express.json());
app.use(cors({ origin: "*" }));

//Test api
app.post("/hello", async (req, res) => {
    return res.status(200).json({ message: "hello" });
})

app.listen(8000);
module.exports = app;
```

执行`npm start`启动

可以通过`http://localhost:8000/hello`查看输出信息

### 18:51 - MongoDB Atlas configuration

https://cloud.mongodb.com/

新建项目`Travel-Story`

Deploy your cluster

Use a template below or set up advanced configuration options. You can also edit these configuration options once the cluster is created.

Add a connection IP address

Create a database user

保存密码

Connecting with MongoDB Driver

创建`config.json`装填`MONGO_DB_URI`

### 23:05 - Creating User schema

```js
import mongoose from "mongoose"

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
  },
  password: {
    type: String,
    required: true,
  },
  createdAt: {
    type: Date,
    default: Date.now(),
  },
})

const User = mongoose.model("User", userSchema)

export default User
```

### 24:26 - Create Account API

执行`node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"`

输出结果如下所示：

```
39cd7527a217f289590110725fa0885fd067cc835eb97bebdbeaa3373658d68ab9164ebf20285bdbc83e0a354e9a2932a0b46c00d02ed5d2a4dd3bed30c320df
```

Error connecting to MongoDB Could not connect to any servers in your MongoDB Atlas cluster. One common reason is that you're trying to access the database from an IP that isn't whitelisted. Make sure your current IP address is on your Atlas cluster's IP whitelist: https://www.mongodb.com/docs/atlas/security-whitelist/

Mongodb开启所有IP

network access 添加 0.0.0.0

### 31:10 - Login API


### 35:02 - Get User API and JWT token authentication

本人在此安装了`cookie-parser`，相应指令为`npm i cookie-parser`

### 40:04 - Add Travel Story API


要在Postman中生成JWT令牌，您可以按照以下步骤操作：

1. 打开Postman应用程序并创建一个新的请求。
2. 在请求中选择“Authorization”选项卡。
3. 在“Type”下拉菜单中选择“Bearer Token”。
4. 在“Token”输入框中输入您的JWT令牌。
5. 单击“Send”按钮，您的请求将包含JWT令牌。

请注意，您需要事先获得JWT令牌才能在Postman中使用它。您可以从您的身份验证服务器或第三方服务中获取JWT令牌。

generateToken.js

```js
import jwt from "jsonwebtoken";

const generateTokenAndSetCookie = (userId, res) => {
	const token = jwt.sign({ userId }, process.env.JWT_SECRET, {
		expiresIn: "15d",
	});

	res.cookie("jwt", token, {
		maxAge: 15 * 24 * 60 * 60 * 1000, // MS
		httpOnly: true, // prevent XSS attacks cross-site scripting attacks
		sameSite: "strict", // CSRF attacks cross-site request forgery attacks
		secure: process.env.NODE_ENV !== "development",
	});
};

export default generateTokenAndSetCookie;
```

在上面的代码中返回或输出token均可，视情况而定。

### 49:05 - Get All Travel Stories API


### 51:07 - Image Upload API using Multer

在`backend`目录下执行`npm i multer`

```
cd backend
xxxxxxxx@DESKTOP-393QBO3:~/TravelStory/backend$ npm i multer

added 16 packages in 3s

23 packages are looking for funding
  run `npm fund` for details
```

Multer是一个用来处理文件上传的Node.js中间件。它允许你轻松地上传文件到服务器。Multer非常灵活，支持处理单个文件上传、多个文件上传，以及各种自定义的上传设置。在Express框架中，Multer可以轻松地与路由处理函数一起使用，使得处理文件上传变得非常简单。它可以处理文件的解析、文件大小的限制、文件类型的限制等等。Multer在Node.js开发中用途广泛，特别是在需要实现文件上传功能的Web应用程序中。


```js
import upload from "../utils/multer.js";
import fs from "fs";
import path from "path";
```


```js
export const imageUpload = async (req, res) => {

	try {
		if (!req.file) {
			return res.status(400).json({ error: true, message: "No image uploaded" })
		}

		const imageUrl = `http://localhost:5000/uploads/${req.file.filename}`;
		res.status(201).json({ imageUrl });
	} catch (error) {
		res.status(500).json({ error: true, message: error.message })
	}
}
```

```js
router.post("/image-upload", upload.single("image"), imageUpload)
```


PostMan无法上传图片，解决办法详见链接。

https://stackoverflow.com/questions/60036239/upload-file-failed-postman

This was what was in my Postman Settings. But in my Desktop, the folder was names as Postman Agent, so the path was ~/Postman Agent/files. Therefore Postman couldn't read from my system, rename Postman Agent in your local machine to Postman.

This is more like a bug with Postman (already open issue, still not corrected).

[![enter image description here](https://i.sstatic.net/Id6pg.png)](https://i.sstatic.net/Id6pg.png)


下面这句代码报错，报错信息为

```
app.use("/uploads", express.static(path.join(__dirname, "uploads")));
                                             ^

ReferenceError: __dirname is not defined in ES module scope
This file is being treated as an ES module because it has a '.js' file extension and '/home/xxxxxxxx/TravelStory/backend/package.json' contains "type": "module". To treat it as a CommonJS script, rename it to use the '.cjs' file extension.
```

有问题的代码如下：

`app.use("/uploads", express.static(path.join(__dirname, "uploads")));`

这该怎么改正？

要解决这个问题，可以将该文件的扩展名从`.js`改为`.cjs`，或者将`package.json`中的`"type": "module"`改为`"type": "commonjs"`。这样就可以在CommonJS模块的范围内使用`__dirname`了。修改后的代码示例如下：

```
app.use("/uploads", express.static(path.join(__dirname, "uploads")));
```

要在ES6模块的范围内使用`__dirname`，可以通过使用`import.meta.url`来动态获取当前模块的路径。修改后的代码如下所示：

```javascript
import { fileURLToPath } from 'url';
import { dirname } from 'path';
import express from 'express';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

app.use("/uploads", express.static(path.join(__dirname, "uploads")));
```

这样就可以在ES6模块中使用`__dirname`来获取当前模块的路径了。

### 58:21 - Delete Image API


### 01:04:15 - Edit Travel Story API


### 01:09:53 - Delete Travel Story API

### 01:13:58 - Update isFavourite API


### 01:18:19 - Search Stories API


### 01:21:58 - Filter Stories by Date Range



## Frontend

### 01:26:12 - Login screen UI


添加images，修改tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    fontFamily: {
      display: ["Poppins", "sans-serif"],
    },
    extend: {
      // Colors used in the project
      colors: {
        primary: "#05B6D3",
        secondary: "#EF863E",
      },
      backgroundImage: {
        'login-bg-img': "url('./src/assets/images/bg-image.png')",
        'signup-bg-img': "url('./src/assets/images/signup-bg-image.jpg')",
      }
    },
  },
  plugins: [],
}

```

在index.css中添加如下内容

```css
@layer components {
  .input-box {
    @apply w-full text-sm bg-cyan-600/5 rounded px-5 py-3 mb-4 outline-none;
  }

  .btn-primary {
    @apply w-full text-sm font-medium text-white bg-cyan-500 shadow-lg shadow-cyan-200/50 p-[10px] rounded-full my-1 hover:bg-cyan-100 hover:text-primary;
  }
}

```

创建passwordInput.jsx文件

安装`npm i react-icons`

新增`utils/helper.js`

### 01:42:28 - Login API integration


在frontend目录下安装axios  `npm i axios`

在`utils`目录下创建`constants.js`和`axiosInstance.js`

### 01:54:14 - Sign-up screen UI


### 01:57:29 - Create Account API integration

### 02:01:57 - Home page UI


### 02:04:55 - Get User Info API integration


在Cards目录下添加`ProfileInfo.jsx`

### 02:08:00 - Navbar Profile Card

### 02:14:20 - Get All Stories API integration

创建emptycard、StoryCard

### 02:17:07 - Travel Story Card component

按照moment 执行`npm i moment`

### 02:26:30 - Function to update isFavourite

安装`react-toastify`执行`npm i react-modal`

安装`react-modal` 执行`npm i react-modal`

### 02:34:14 - Add/Edit Travel Story

在`index.css`中添加

```css
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: "Poppins", "sans-serif";
  }

  body {
    background-color: #fdfeff;
    overflow-x: hidden;
  }
}

/* Customize scrollbar styles*/
.scrollbar::-webkit-scrollbar {
  width: 3px;
  height: 3px;
}

.scrollbar::-webkit-scrollbar-thumb {
  background-color: #057c8e;
  border-radius: 3px;
}

.scrollbar::-webkit-scrollbar-track {
  background-color: rgb(172, 201, 229);
}

@layer components {
  .input-box {
    @apply w-full text-sm bg-cyan-600/5 rounded px-5 py-3 mb-4 outline-none;
  }

  .btn-primary {
    @apply w-full text-sm font-medium text-white bg-cyan-500 shadow-lg shadow-cyan-200/50 p-[10px] rounded-full my-1 hover:bg-cyan-100 hover:text-primary;
  }

  .login-ui-box {
    @apply w-80 h-[450px] rounded-full bg-primary absolute rotate-45
  }

  .model-box {
    @apply w-[80vw] md:w-[40%] h-[80vh] bg-white rounded-lg mx-auto mt-14 p-5 overflow-y-scroll scrollbar z-50;
  }

  .icon-btn {
    @apply text-[22px] text-slate-300 cursor-pointer hover:text-red-500;
  }

  .btn-small {
    @apply flex items-center gap-1 text-xs font-medium bg-cyan-50 text-primary shadow-cyan-100/0 border border-cyan-100 hover:bg-primary hover:text-white rounded px-3 py-[3px];
  }
  .input-label {
    @apply text-xs text-slate-400;
  }
  .btn-delete {
    @apply bg-rose-50 text-rose-500 shadow-cyan-100/0 border border-rose-100 hover:border-rose-500 hover:text-white;
  }
}
```

新增Input目录下`DateSelector`

### 02:47:12 - Date Selector component

安装`react-day-picker` 执行`npm i react-day-picker`

在`main.jsx`中添加`import "react-day-picker/style.css";`

在`index.css`中添加

```css
.rdp-root {
    --rdp-accent-color: #01b0cb;
    --rdp-accent-background-color: #dffbff;
    --rdp-day_button-border-radius: 8px;
    --rdp-selected-font: bold medium var(--rdp-font-family);
  }
```

### 02:58:34 - Custom Image Picker component

增加`ImageSelector`组件

添加`TagInput`组件

### 03:22:10 - Function to add new Travel Story


### 03:24:34 - Utility function to upload image

在`utils`下面添加`uploadImage.js`

### 03:31:43 - View Travel Story popup modal

在home目录下添加`ViewTravelStory.jsx`

### 03:45:11 - Function to update story

### 03:51:20 - Function to delete Travel Story image

### 03:58:37 - Function to delete story

### 04:09:54 - Search Bar component

### 04:14:05 - Search Stories API integration

### 04:17:59 - Date Range Picker component

### 04:21:31 - Filter Travel Stories by date range

### 04:25:35 - Filter Info Title component

在`Cards`下新建`FilterInfoTitle.jsx`


## 附录

### 没用上的`utilities.js`

```js
const jwt = require('jsonwebtoken')

function authenticateToken(req, res, next) {
    const authHeader = req.headers["authorization"];
    const token = authHeader && authHeader.split[" "][1];

    // No token, unauthorized
    if (!token) return res.sendState(401);
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        // Token invalid, forbidden
        if(err) return res.sendStatus(401);
        req.user = user;
        next();
    });
}

module.exports = {
    authenticateToken,
};
```

### 自用需要配置`.env`

```
PORT=xxxx

JWT_SECRET=xxxxxxxxxxx

MONGO_DB_URI=xxxxxxx

NODE_ENV=xxxxxx
```


### [Wiki: bcrypt vs brypt.js](https://github.com/kelektiv/node.bcrypt.js/wiki/bcrypt-vs-brypt.js)

Short summary: `bcrypt` is a native (C++) module, thus much faster than `bcryptjs` which is a pure js module.

`bcrypt` sometimes requires additional steps to build correctly, especially if you are using architectures other than x86_64 or a glibc based distro. You will need additional dependencies to compile from source.

`bcryptjs` is plain js, hence works everywhere, even browsers. `bcrypt` runs only on NodeJS, Node-WebKit or Electron.

### Free SVG converter

https://www.freeconvert.com/jpg-to-svg/download

https://picsvg.com/

[Picsvg](https://picsvg.com/)

Need to convert a picture to SVG format ?

Picsvg is a free online converter that can convert an image to a SVG file.You can upload an image file (jpg,gif,png) up to 4 Mb, then you can select effects to enhance the SVG image result.















