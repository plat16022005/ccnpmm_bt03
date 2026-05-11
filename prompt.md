**HƯỚNG DẪN FULLSTACK**

Express.js - ReactJS - Mongoose

CÔNG NGHỆ PHẦN MỀM MỚI (MTSE431179)

**Giảng viên:** ThS. Nguyễn Hữu Trung

Khoa Công Nghệ Thông Tin - Trường Đại học Sư Phạm Kỹ Thuật TP.HCM

Email: <trungnh@hcmute.edu.vn> | SĐT: 090.861.7108

YouTube: <https://www.youtube.com/@baigiai>

# **NỘI DUNG BÀI HỌC**

- Phần 1: Hướng dẫn xây dựng BackEnd API với ExpressJS
- Phần 2: Hướng dẫn xây dựng FrontEnd với ReactJS

# **PHẦN 1: XÂY DỰNG BACKEND API VỚI EXPRESSJS**

**Bước 1:** Cài đặt môi trường và khởi tạo dự án

Cài đặt node.js (>18.), MongoDB Compass, tạo thư mục FullStackNodeJS01\\ExpressJS01 và FullStackNodeJS01\\ReactJS01.

Mở thư mục FullStackNodeJS01 trong Explorer → gõ cmd trên thanh địa chỉ → gõ code . để mở VSCode.

Trong Terminal của VSCode, gõ lệnh:

**Lệnh khởi tạo dự án BackEnd:**

cd ExpressJS01

npm init

**Nội dung file ExpressJS01\\package.json sau khi khởi tạo:**

{

"name": "expressjs01",

"version": "1.0.0",

"description": "Backend API for FullStack",

"main": "server.js",

"scripts": {

"test": "echo \\"Error: no test specified\\" && exit 1"

},

"keywords": \["nodejs", "expressjs", "reactjs"\],

"author": "Nguyễn Hữu Trung",

"license": "ISC"

}

**Bước 2:** Cài đặt Express.js, Mongoose và các thư viện cần thiết

**Cài dependencies:**

npm install --save express mongoose dotenv ejs cors bcrypt jsonwebtoken

**Cài devDependencies:**

npm install --save-dev @babel/core @babel/node @babel/preset-env nodemon

**Thêm vào thẻ "scripts" trong package.json:**

"scripts": {

"dev": "nodemon ./src/server.js",

"start": "nodemon ./src/server.js"

},

**Bước 3:** Tạo cấu trúc thư mục và các file cơ bản

**Cài thêm type definitions:**

npm i --save-dev @types/cors

npm i --save-dev @types/express

**Cấu trúc thư mục dự án ExpressJS01:**

FULLSTACKNODEJS01/

└── ExpressJS01/

├── node_modules/

├── src/

│ ├── config/

│ │ ├── database.js

│ │ └── viewEngine.js

│ ├── controllers/

│ │ ├── homeController.js

│ │ └── userController.js

│ ├── middleware/

│ │ ├── auth.js

│ │ └── delay.js

│ ├── models/

│ │ └── user.js

│ ├── public/

│ ├── routes/

│ │ └── api.js

│ ├── services/

│ │ └── userService.js

│ ├── views/

│ │ └── index.ejs

│ └── server.js

├── .env

├── .gitignore

├── package-lock.json

└── package.json

**Nội dung file .env:**

NODE_ENV=development

PORT=8080

MONGO_DB_URL=mongodb://localhost:27017/fullstack02

JWT_SECRET=your_jwt_secret_key

JWT_EXPIRE=1d

**Nội dung file .gitignore:**

node_modules

\# .env

**Bước 4:** Tạo thư mục src\\config và các file cấu hình

**File: src/config/viewEngine.js**

const path = require('path');

const express = require('express');

const configViewEngine = (app) => {

app.set('views', path.join('./src', 'views'));

app.set('view engine', 'ejs');

//config static files: image/css/js

app.use(express.static(path.join('./src', 'public')));

}

module.exports = configViewEngine;

**File: src/config/database.js**

require('dotenv').config();

const mongoose = require('mongoose');

const dbState = \[{

value: 0,

label: 'Disconnected'

}, {

value: 1,

label: 'Connected'

}, {

value: 2,

label: 'Connecting'

}, {

value: 3,

label: 'Disconnecting'

}\];

const connection = async () => {

await mongoose.connect(process.env.MONGO_DB_URL);

const state = Number(mongoose.connection.readyState);

console.log(dbState.find(f => f.value === state).label, 'to database');

}

module.exports = connection;

**Bước 5:** Cấu hình server.js

**File: src/server.js**

require('dotenv').config();

//import các nguồn cần dùng

const express = require('express'); //commonjs

const configViewEngine = require('./config/viewEngine');

const apiRoutes = require('./routes/api');

const connection = require('./config/database');

const { getHomepage } = require('./controllers/homeController');

const cors = require('cors');

const app = express(); //cấu hình app là express

//cấu hình port, nếu tìm thấy port trong env, không thì trả về 8888

const port = process.env.PORT || 8888;

app.use(cors()); //config cors

app.use(express.json()) // //config req.body cho json

app.use(express.urlencoded({ extended: true })) // for form data

configViewEngine(app); //config template engine

//config route cho view ejs

const webAPI = express.Router();

webAPI.get('/', getHomepage);

app.use('/', webAPI);

//khai báo route cho API

app.use('/v1/api/', apiRoutes);

(async () => {

try {

//kết nối database using mongoose

await connection();

//lắng nghe port trong env

app.listen(port, () => {

console.log(\`Backend Nodejs App listening on port \${port}\`)

})

} catch (error) {

console.log('>>> Error connect to DB: ', error)

}

})();

**Bước 6:** Tạo file api.js trong thư mục src\\routes

**File: src/routes/api.js**

const express = require('express');

const { createUser, handleLogin, getUser,

getAccount

} = require('../controllers/userController');

const auth = require('../middleware/auth');

const delay = require('../middleware/delay');

const routerAPI = express.Router();

routerAPI.all('\*', auth);

routerAPI.get('/', (req, res) => {

return res.status(200).json('Hello world api')

})

routerAPI.post('/register', createUser);

routerAPI.post('/login', handleLogin);

routerAPI.get('/user', getUser);

routerAPI.get('/account', delay, getAccount);

module.exports = routerAPI; //export default

**Bước 7:** Tạo file user.js trong thư mục src\\models

**File: src/models/user.js**

const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({

name: String,

email: String,

password: String,

role: String,

});

const User = mongoose.model('user', userSchema);

module.exports = User;

**Bước 8:** Tạo file userService.js trong thư mục src\\services

**File: src/services/userService.js**

require('dotenv').config();

const User = require('../models/user');

const bcrypt = require('bcrypt');

const jwt = require('jsonwebtoken');

const saltRounds = 10;

const createUserService = async (name, email, password) => {

try {

//check user exist

const user = await User.findOne({ email });

if (user) {

console.log(\`>>> user exist, chọn 1 email khác: \${email}\`);

return null;

}

//hash user password

const hashPassword = await bcrypt.hash(password, saltRounds)

//save user to database

let result = await User.create({

name: name,

email: email,

password: hashPassword,

role: 'User'

})

return result;

} catch (error) {

console.log(error);

return null;

}

}

const loginService = async (email1, password) => {

try {

//fetch user by email

const user = await User.findOne({ email: email1 });

if (user) {

//compare password

const isMatchPassword = await bcrypt.compare(password, user.password);

if (!isMatchPassword) {

return {

EC: 2,

EM: 'Email/Password không hợp lệ'

}

} else {

//create an access token

const payload = {

email: user.email,

name: user.name

}

const access_token = jwt.sign(

payload,

process.env.JWT_SECRET,

{

expiresIn: process.env.JWT_EXPIRE

}

)

return {

EC: 0,

access_token,

user: {

email: user.email,

name: user.name

}

};

}

} else {

return {

EC: 1,

EM: 'Email/Password không hợp lệ'

}

}

} catch (error) {

console.log(error);

return null;

}

}

const getUserService = async () => {

try {

let result = await User.find({}).select('-password');

return result;

} catch (error) {

console.log(error);

return null;

}

}

module.exports = {

createUserService, loginService, getUserService

}

**Bước 9:** Tạo homeController.js, userController.js và views/index.ejs

**File: src/controllers/homeController.js**

const getHomepage = async (req, res) => {

return res.render('index.ejs')

}

module.exports = {

getHomepage,

}

**File: src/views/index.ejs**

&lt;html lang="en"&gt;

&lt;head&gt;

&lt;meta charset="UTF-8"&gt;

&lt;meta http-equiv="X-UA-Compatible" content="IE=edge"&gt;

&lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;

&lt;title&gt;Nodejs&lt;/title&gt;

&lt;/head&gt;

&lt;body&gt;

&lt;h1&gt; hello world with nodejs&lt;/h1&gt;

&lt;/body&gt;

&lt;/html&gt;

**File: src/controllers/userController.js**

const { createUserService, loginService, getUserService } = require('../services/userService');

const createUser = async (req, res) => {

const { name, email, password } = req.body;

const data = await createUserService(name, email, password);

return res.status(200).json(data)

}

const handleLogin = async (req, res) => {

const { email, password } = req.body;

const data = await loginService(email, password);

return res.status(200).json(data)

}

const getUser = async (req, res) => {

const data = await getUserService();

return res.status(200).json(data)

}

const getAccount = async (req, res) => {

return res.status(200).json(req.user)

}

module.exports = {

createUser, handleLogin, getUser, getAccount

}

**Bước 10:** Tạo file auth.js và delay.js trong thư mục src\\middleware

**File: src/middleware/auth.js**

require('dotenv').config();

const jwt = require('jsonwebtoken');

const auth = (req, res, next) => {

const white_lists = \['/', '/register', '/login'\];

if (white_lists.find(item => '/v1/api' + item === req.originalUrl)) {

next();

} else {

if (req?.headers?.authorization?.split(' ')?.\[1\]) {

const token = req.headers.authorization.split(' ')\[1\];

//verify token

try {

const decoded = jwt.verify(token, process.env.JWT_SECRET);

req.user = {

email: decoded.email,

name: decoded.name,

createdBy: 'hoidanit'

}

console.log('>>> check token: ', decoded)

next();

} catch (error) {

return res.status(401).json({

message: 'Token bị hết hạn/hoặc không hợp lệ'

})

}

} else {

return res.status(401).json({

message: 'Bạn chưa truyền Access Token ở header/Hoặc token bị hết hạn'

})

}

}

}

module.exports = auth;

**File: src/middleware/delay.js**

const delay = (req, res, next) => {

setTimeout(() => {

if (req.headers.authorization) {

const token = req.headers.authorization.split(' ')\[1\];

console.log('>>> check token: ', token)

}

next()

}, 3000)

}

module.exports = delay;

**Bước 11:** Chạy dự án BackEnd

**Lệnh chạy server:**

npm start

Kết quả hiển thị trong terminal khi server khởi động thành công:

\[nodemon\] 3.1.4

\[nodemon\] to restart at any time, enter \`rs\`

\[nodemon\] watching path(s): \*.\*

\[nodemon\] watching extensions: js,mjs,cjs,json

\[nodemon\] starting \`node ./src/server.js\`

Connected to database

Backend Nodejs App listening on port 8080

**Bước 12:** Test API bằng Postman

Sử dụng Postman để kiểm tra các endpoint API:

**1\. Register user (POST <http://localhost:8080/v1/api/register>):**

Body (x-www-form-urlencoded):

name = Hữu Trung

email = <trungnh2@hcmute.edu.vn>

password = 123456

Response thành công (Status 200 OK):

{

"name": "Hữu Trung",

"email": "<trungnh2@hcmute.edu.vn>",

"password": "\$2b\$10\$ozj1n6GCuxn3UJc42SnX/.CFd1s8weeZRZPARgHTNJef1gdo0qrta",

"role": "User",

"\_id": "66b1a00f8168d49adfea80fe",

"\_\_v": 0

}

**2\. Login user (POST <http://localhost:8080/v1/api/login>):**

Body (x-www-form-urlencoded):

email = <trungnh1@hcmute.edu.vn>

password = 123456

Response thành công (Status 200 OK):

{

"EC": 0,

"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"user": {

"email": "<trungnh1@hcmute.edu.vn>",

"name": "Nguyễn Hữu Trung"

}

}

**3\. Test Homepage với Token (GET <http://localhost:8080/v1/api>):**

Authorization: Bearer Token → dán access_token nhận được từ login

Response thành công (Status 200 OK):

"Nguyễn Hữu Trung! Hello world! HomePage API"

# **PHẦN 2: XÂY DỰNG FRONTEND VỚI REACTJS**

**Bước 13:** Khởi tạo dự án ReactJS với ViteJS

**Tại VSCode, mở Terminal → New Terminal, rồi chạy lệnh:**

cd ReactJS01

npm create vite@latest

\# Nhập tên project: reactjs01

\# Chọn: React

\# Chọn: JavaScript + SWC

cd reactjs01

npm install

npm run dev

\# Build & deploy:

npm run build

**Bước 14:** Cài đặt các thư viện cần thiết cho FrontEnd

**Cài dependencies cho ReactJS:**

npm install --save react-router-dom axios antd @ant-design/icons

**Thêm vào thẻ "scripts" trong package.json:**

"scripts": {

"dev": "vite",

"start": "vite",

"build": "vite build",

"lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0",

"preview": "vite preview"

},

**Bước 15:** Cấu trúc thư mục dự án ReactJS

**Cấu trúc thư mục ReactJS01/reactjs01:**

ReactJS01/

└── reactjs01/

├── node_modules/

├── public/

├── src/

│ ├── assets/

│ ├── components/

│ │ ├── context/

│ │ │ └── auth.context.jsx

│ │ └── layout/

│ │ └── header.jsx

│ ├── pages/

│ │ ├── home.jsx

│ │ ├── login.jsx

│ │ ├── register.jsx

│ │ └── user.jsx

│ ├── styles/

│ │ └── global.css

│ ├── util/

│ │ ├── api.js

│ │ └── axios.customize.js

│ ├── App.jsx

│ └── main.jsx

├── .env.development

├── .env.production

├── .eslintrc.cjs

├── .gitignore

├── index.html

├── package-lock.json

├── package.json

└── vite.config.js

**File: vite.config.js**

import { defineConfig } from 'vite'

import react from '@vitejs/plugin-react-swc'

// <https://vitejs.dev/config/>

export default defineConfig({

plugins: \[react()\],

})

**File: .env.development**

VITE_BACKEND_URL=<http://localhost:8080>

**File: index.html**

&lt;!doctype html&gt;

&lt;html lang="en"&gt;

&lt;head&gt;

&lt;meta charset="UTF-8" /&gt;

&lt;link rel="icon" type="image/svg+xml" href="/vite.svg" /&gt;

&lt;meta name="viewport" content="width=device-width, initial-scale=1.0" /&gt;

&lt;title&gt;FrontEnd Nguyễn Hữu Trung&lt;/title&gt;

&lt;/head&gt;

&lt;body&gt;

&lt;div id="root"&gt;&lt;/div&gt;

&lt;script type="module" src="/src/main.jsx"&gt;&lt;/script&gt;

&lt;/body&gt;

&lt;/html&gt;

**Bước 16:** Tạo thư mục src\\components và các file context, layout

**File: src/components/context/auth.context.jsx**

import { createContext, useState } from 'react';

export const AuthContext = createContext({

isAuthenticated: false,

user: {

email: "",

name: ""

},

appLoading: true,

});

export const AuthWrapper = (props) => {

const \[auth, setAuth\] = useState({

isAuthenticated: false,

user: {

email: "",

name: ""

}

});

const \[appLoading, setAppLoading\] = useState(true);

return (

<AuthContext.Provider value={{

auth, setAuth, appLoading, setAppLoading

}}>

{props.children}

&lt;/AuthContext.Provider&gt;

);

}

**File: src/components/layout/header.jsx**

import React, { useContext, useState } from 'react';

import { UsergroupAddOutlined, HomeOutlined, SettingOutlined } from '@ant-design/icons';

import { Menu } from 'antd';

import { Link, useNavigate } from 'react-router-dom';

import { AuthContext } from '../context/auth.context';

const Header = () => {

const navigate = useNavigate();

const { auth, setAuth } = useContext(AuthContext);

console.log('>>> check auth: ', auth)

const items = \[

{

label: &lt;Link to={'/'}&gt;Home Page&lt;/Link&gt;,

key: 'home',

icon: &lt;HomeOutlined /&gt;,

},

...(auth.isAuthenticated ? \[{

label: &lt;Link to={'/user'}&gt;Users&lt;/Link&gt;,

key: 'user',

icon: &lt;UsergroupAddOutlined /&gt;,

}\] : \[\]),

{

label: \`Welcome \${auth?.user?.email ?? ''}\`,

key: 'SubMenu',

icon: &lt;SettingOutlined /&gt;,

children: \[

...(auth.isAuthenticated ? \[{

label: &lt;span onClick={() =&gt; {

localStorage.clear('access_token');

setAuth({

isAuthenticated: false,

user: {

email: "",

name: ""

}

})

navigate('/');

}}>Đăng xuất&lt;/span&gt;,

key: 'logout',

}\] : \[

{

label: &lt;Link to={'/login'}&gt;Đăng nhập&lt;/Link&gt;,

key: 'login',

}

\]),

\],

},

\];

const \[current, setCurrent\] = useState('mail');

const onClick = (e) => {

console.log('click ', e);

setCurrent(e.key);

};

return &lt;Menu onClick={onClick} selectedKeys={\[current\]} mode="horizontal" items={items} /&gt;;

};

export default Header;

**Bước 17:** Tạo src\\util, src\\styles, src\\pages và các file

**File: src/util/axios.customize.js**

import axios from 'axios';

// Set config defaults when creating the instance

const instance = axios.create({

baseURL: import.meta.env.VITE_BACKEND_URL

});

// Alter defaults after instance has been created

// Add a request interceptor

instance.interceptors.request.use(function (config) {

// Do something before request is sent

config.headers.Authorization = \`Bearer \${localStorage.getItem('access_token')}\`;

return config;

}, function (error) {

// Do something with request error

return Promise.reject(error);

});

// Add a response interceptor

instance.interceptors.response.use(function (response) {

// Any status code that lie within the range of 2xx cause this function to trigger

// Do something with response data

if (response && response.data) return response.data;

return response;

}, function (error) {

// Any status codes that falls outside the range of 2xx cause this function to trigger

// Do something with response error

if (error?.response?.data) return error?.response?.data;

return Promise.reject(error);

});

export default instance;

**File: src/util/api.js**

import axios from './axios.customize';

const createUserApi = (name, email, password) => {

const URL_API = '/v1/api/register';

const data = {

name, email, password

}

return axios.post(URL_API, data)

}

const loginApi = (email, password) => {

const URL_API = '/v1/api/login';

const data = {

email, password

}

return axios.post(URL_API, data)

}

const getUserApi = () => {

const URL_API = '/v1/api/user';

return axios.get(URL_API)

}

export {

createUserApi, loginApi, getUserApi

}

**File: src/styles/global.css**

\* {

margin: 0;

padding: 0;

}

**File: src/pages/home.jsx**

import { CrownOutlined } from '@ant-design/icons';

import { Result } from 'antd';

const HomePage = () => {

return (

&lt;div style={{ padding: 20 }}&gt;

<Result

icon={&lt;CrownOutlined /&gt;}

title="JSON Web Token (React/Node.JS) - iotstar.vn"

/>

&lt;/div&gt;

)

}

export default HomePage;

**File: src/pages/user.jsx**

import { notification, Table } from 'antd';

import { useEffect, useState } from 'react';

import { getUserApi } from '../util/api';

const UserPage = () => {

const \[dataSource, setDataSource\] = useState(\[\]);

useEffect(() => {

const fetchUser = async () => {

const res = await getUserApi();

if (!res?.message) {

setDataSource(res)

} else {

notification.error({

message: 'Unauthorized',

description: res.message

})

}

}

fetchUser();

}, \[\])

const columns = \[

{ title: 'Id', dataIndex: '\_id' },

{ title: 'Email', dataIndex: 'email' },

{ title: 'Name', dataIndex: 'name' },

{ title: 'Role', dataIndex: 'role' },

\];

return (

&lt;div style={{ padding: 30 }}&gt;

<Table

bordered

dataSource={dataSource}

columns={columns}

rowKey={'\_id'}

/>

&lt;/div&gt;

)

}

export default UserPage;

**File: src/pages/register.jsx**

import React from 'react';

import { Button, Col, Divider, Form, Input, notification, Row } from 'antd';

import { createUserApi } from '../util/api';

import { Link, useNavigate } from 'react-router-dom';

import { ArrowLeftOutlined } from '@ant-design/icons';

const RegisterPage = () => {

const navigate = useNavigate();

const onFinish = async (values) => {

const { name, email, password } = values;

const res = await createUserApi(name, email, password);

if (res) {

notification.success({

message: 'CREATE USER',

description: 'Success'

});

navigate('/login');

} else {

notification.error({

message: 'CREATE USER',

description: 'error'

})

}

};

return (

&lt;Row justify={'center'} style={{ marginTop: '30px' }}&gt;

&lt;Col xs={24} md={16} lg={8}&gt;

<fieldset style={{

padding: '15px',

margin: '5px',

border: '1px solid #ccc',

borderRadius: '5px'

}}>

&lt;legend&gt;Đăng Ký Tài Khoản&lt;/legend&gt;

<Form

name='basic'

onFinish={onFinish}

autoComplete='off'

layout='vertical'

\>

<Form.Item label='Email' name='email'

rules={\[{ required: true, message: 'Please input your email!' }\]}

\>

&lt;Input /&gt;

&lt;/Form.Item&gt;

<Form.Item label='Password' name='password'

rules={\[{ required: true, message: 'Please input your password!' }\]}

\>

&lt;Input.Password /&gt;

&lt;/Form.Item&gt;

<Form.Item label='Name' name='name'

rules={\[{ required: true, message: 'Please input your name!' }\]}

\>

&lt;Input /&gt;

&lt;/Form.Item&gt;

&lt;Form.Item&gt;

&lt;Button type='primary' htmlType='submit'&gt;Submit&lt;/Button&gt;

&lt;/Form.Item&gt;

&lt;/Form&gt;

&lt;Link to={'/'}&gt;&lt;ArrowLeftOutlined /&gt; Quay lại trang chủ&lt;/Link&gt;

&lt;Divider /&gt;

&lt;div style={{ textAlign: 'center' }}&gt;

Đã có tài khoản? &lt;Link to={'/login'}&gt;Đăng nhập&lt;/Link&gt;

&lt;/div&gt;

&lt;/fieldset&gt;

&lt;/Col&gt;

&lt;/Row&gt;

)

}

export default RegisterPage;

**File: src/pages/login.jsx**

import React, { useContext } from 'react';

import { Button, Col, Divider, Form, Input, notification, Row } from 'antd';

import { loginApi } from '../util/api';

import { Link, useNavigate } from 'react-router-dom';

import { AuthContext } from '../components/context/auth.context';

import { ArrowLeftOutlined } from '@ant-design/icons';

const LoginPage = () => {

const navigate = useNavigate();

const { setAuth } = useContext(AuthContext);

const onFinish = async (values) => {

const { email, password } = values;

const res = await loginApi(email, password);

if (res && res.EC === 0) {

localStorage.setItem('access_token', res.access_token)

notification.success({

message: 'LOGIN USER',

description: 'Success'

});

setAuth({

isAuthenticated: true,

user: {

email: res?.user?.email ?? '',

name: res?.user?.name ?? ''

}

})

navigate('/');

} else {

notification.error({

message: 'LOGIN USER',

description: res?.EM ?? 'error'

})

}

};

return (

&lt;Row justify={'center'} style={{ marginTop: '30px' }}&gt;

&lt;Col xs={24} md={16} lg={8}&gt;

<fieldset style={{

padding: '15px',

margin: '5px',

border: '1px solid #ccc',

borderRadius: '5px'

}}>

&lt;legend&gt;Đăng Nhập&lt;/legend&gt;

<Form

name='basic'

onFinish={onFinish}

autoComplete='off'

layout='vertical'

\>

<Form.Item label='Email' name='email'

rules={\[{ required: true, message: 'Please input your email!' }\]}

\>

&lt;Input /&gt;

&lt;/Form.Item&gt;

<Form.Item label='Password' name='password'

rules={\[{ required: true, message: 'Please input your password!' }\]}

\>

&lt;Input.Password /&gt;

&lt;/Form.Item&gt;

&lt;Form.Item&gt;

&lt;Button type='primary' htmlType='submit'&gt;Login&lt;/Button&gt;

&lt;/Form.Item&gt;

&lt;/Form&gt;

&lt;Link to={'/'}&gt;&lt;ArrowLeftOutlined /&gt; Quay lại trang chủ&lt;/Link&gt;

&lt;Divider /&gt;

&lt;div style={{ textAlign: 'center' }}&gt;

Chưa có tài khoản? &lt;Link to={'/register'}&gt;Đăng ký tại đây&lt;/Link&gt;

&lt;/div&gt;

&lt;/fieldset&gt;

&lt;/Col&gt;

&lt;/Row&gt;

)

}

export default LoginPage;

**Bước 18:** Cấu hình main.jsx và App.jsx

**File: src/main.jsx**

import React from 'react'

import ReactDOM from 'react-dom/client'

import App from './App.jsx'

import './styles/global.css';

import {

createBrowserRouter,

RouterProvider,

} from 'react-router-dom';

import RegisterPage from './pages/register.jsx';

import UserPage from './pages/user.jsx';

import HomePage from './pages/home.jsx';

import LoginPage from './pages/login.jsx';

import { AuthWrapper } from './components/context/auth.context.jsx';

const router = createBrowserRouter(\[

{

path: '/',

element: &lt;App /&gt;,

children: \[

{

index: true,

element: &lt;HomePage /&gt;

},

{

path: 'user',

element: &lt;UserPage /&gt;

},

{

path: 'register',

element: &lt;RegisterPage /&gt;

},

{

path: 'login',

element: &lt;LoginPage /&gt;

},

\]

},

\]);

ReactDOM.createRoot(document.getElementById('root')).render(

&lt;React.StrictMode&gt;

&lt;AuthWrapper&gt;

&lt;RouterProvider router={router} /&gt;

&lt;/AuthWrapper&gt;

&lt;/React.StrictMode&gt;,

)

**File: src/App.jsx**

import { Outlet } from 'react-router-dom';

import Header from './components/layout/header';

import axios from './util/axios.customize'

import { useContext, useEffect } from 'react'

import { AuthContext } from './components/context/auth.context';

import { Spin } from 'antd';

function App() {

const { setAuth, appLoading, setAppLoading } = useContext(AuthContext);

useEffect(() => {

const fetchAccount = async () => {

setAppLoading(true);

const res = await axios.get(\`/v1/api/user\`);

if (res && !res.message) {

setAuth({

isAuthenticated: true,

user: {

email: res.email,

name: res.name

}

})

}

setAppLoading(false);

}

fetchAccount()

}, \[\])

return (

&lt;div&gt;

{appLoading === true ?

<div style={{

position: 'fixed',

top: '50%',

left: '50%',

transform: 'translate(-50%, -50%)'

}}>

&lt;Spin /&gt;

&lt;/div&gt;

:

<>

&lt;Header /&gt;

&lt;Outlet /&gt;

&lt;/&gt;

}

&lt;/div&gt;

)

}

export default App

# **BÀI TẬP**

Thực hiện các bước tương tự như bài tập hướng dẫn nhưng áp dụng cho database MySQL.

- Thực hiện chức năng Register và ForgotPassword

Gợi ý: Thay Mongoose bằng thư viện mysql2 hoặc sequelize để kết nối MySQL.