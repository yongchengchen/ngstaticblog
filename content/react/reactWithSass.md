<!--
Categories = ["Development", "react"]
Description = "Hook Scripts help to Add SASS/SCSS to a create-react-app Project"
Tags = ["Development", "react", "scss", "sass", "create-react-app"]
date = "2018-01-25T21:47:31-08:00"
title = "Hook Scripts help to Add SASS/SCSS to a create-react-app Project"
-->

## Hook Scripts help to Add SASS/SCSS to a create-react-app Project

Just implemented a very simple hook scripts which help to add SASS/SCSS to a create-react-app Project. It's very easy to  add the SASS/SCSS precompiler to an existing create-react-app project. 

### Installation

#### 1. Install support modules

Via yarn
```shell
yarn add sass-loader node-sass style-loader css-loader
```
Via npm
```shell
yarn install sass-loader node-sass style-loader css-loader --save-dev
```

#### 2. Checkout hook scripts
```shell
cd your_react_app_folder
git clone https://github.com/yongchengchen/react-script-scss-hook.git hooks
```

#### 3. Add script trigger to "scripts" node of package.json
```shell
"prestart": "node hooks/pre.js",
"prebuild": "node hooks/pre.js",
```

### Usage

#### 1. import scss to your component

```js
    import './style.scss'
```

#### 2. build or start it

```shell
yarn start

//or

yarn build
```
