# 自己的函式庫，也能用 npm install 安裝



* 事前工作
    * 帳號(預備)
        * github 放 source code 的地方
        * npm 帳號, 最後 push 上去要用到
            * 若還沒有 npm 帳號，先申請 https://www.npmjs.com/signup
            * 推送會用到 username，登入時
    * 軟體 (vscode)
    * 概念
        * 一個專案，在電腦是一個資料夾。
        * 承上，資料夾名稱，通常是專案名稱。
        * 承上，專案名稱通常是"小寫"與"-"組成。
        * 例: ijn-fhl-sharefun-ts


# 網頁內容 
參考網頁:
https://itnext.io/step-by-step-building-and-publishing-an-npm-typescript-package-44fe7164964c

## 新增專案，並將專案與 github 相連結

```bash=
echo "# tech-npm-packages-a" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/git帳號/專案名稱.git
git push -u origin main
# 例: git remote add origin https://github.com/snowray712000/tech-npm-packages-a.git
```

## 加入「需用」套件
typescript prettier tslint jest
### 操作
將下面全選，貼在 terminate 中執行即可
```bash=
# 新增 package.json
npm init -y

# 新增 .gitignore
echo "node_modules" >> .gitignore
echo "/lib" >> .gitignore

# 新增 typescript 及 prettier tslint
npm install --save-dev typescript
echo '{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./lib",
    "strict": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "**/__tests__/*"]
}' >> tsconfig.json

npm install --save-dev prettier tslint tslint-config-prettier
echo '{
   "extends": ["tslint:recommended", "tslint-config-prettier"]
}' >> tslint.json
echo '{
  "printWidth": 120,
  "trailingComma": "all",
  "singleQuote": true
}' >> .prettierrc

# 新增 jest，測試專案用 
npm install --save-dev jest ts-jest @types/jest
echo '{
  "transform": {
    "^.+\\\\.(t|j)sx?$": "ts-jest"
  },
  "testRegex": "(/__tests__/.*|(\\\\.|/)(test|spec))\\\\.(jsx?|tsx?)$",
  "moduleFileExtensions": ["ts", "tsx", "js", "jsx", "json", "node"]
}' >> jestconfig.json
```
### 補充說明(不看也行)
* echo 指令
    * echo 指令就是將"文字"寫進檔案尾部。
    * echo 可以用單引號、也可以用雙引號。
    * echo 有些字元要跳脫，如\`、\'、\"、\\
    * echo 可以包含換行
* --save-dev 參數
    * devDependency。表示這個套件，只有「開發者」會用到，「使用者」不需要用到。
* .gitignore
    * 不推到 github 上。
    * node_modules， 內容在網路上就有，在 package.json 有記載套件及版本，所以只有有網頁，仍然可以還原，所以不用把 node_modules 的東西推到 git 中
    * /lib，這是 compile 後，會產出的資料夾，所以也不用推到 git 中。順帶一提，這個在 'tsconfig.json' 檔中設定。
* typescript 套件
    * 一種程式語言，相較於 javascript。
    * 概念...它最後寫出來的時候，compile後，就是變成 javascript{給網頁用}。
* tslint prettierrc 套件
    * 檢查程式碼排版要求(一開始會覺得很綁手綁腳)
* jest 測試套件
    * facebook 用的測試套件

## 手動微調 package.json
### 操作
將下面貼在 package.json 最下面
然後將上面重複的取代掉
因為下面只有 types 與 files 是原本沒有的
```json=
{
    "scripts": {
       "test": "jest --config jestconfig.json",
       "build": "tsc",
       "format": "prettier --write \"src/**/*.ts\" \"src/**/*.js\"",
       "lint": "tslint -p tsconfig.json",
       "prepare": "npm run build",
       "prepublishOnly": "npm test && npm run lint",
       "preversion": "npm run lint",
       "version": "npm run format && git add -A src",
       "postversion": "git push && git push --tags"
    },   
   "main": "lib/index.js",
   "types": "lib/index.d.ts",
   "description": "對此專案的描述",
   "keywords": ["關鍵字1", "關鍵字2"],
   "author": "作者名稱",
   "files": ["lib/**/*"]
}

```
### 補充說明
* scripts
    * 在 terminate 執行 'npm run build' 就等於執行 tsc
    * 執行 npm run test 就等於執行 jest --config jestconfig.json
    * prepare 之後那幾個 ... 是推到 npm、會自動依序被執行的。
* main
    * 這個 {編譯好的} lib 進入點是哪個檔案
* types
    * {編譯}同時輸入宣告檔(declare)，即 .d.ts。這樣，typescript專案也可以引用此 lib了。
* files
    * 指定 publish 到 npm 時，哪些要丟給人家使用。
## 程式碼範例
### 操作_建立一函式與測試
建立範例 code 與對應的 test code
```bash=
mkdir src && mkdir src/__tests__
echo "export const Greeter = (name: string) => \`Hello \${name}\`; " >> src/index.ts

echo "import { Greeter } from '../index';
  
  test('My Greeter', () => {
    expect(Greeter('Carl')).toBe('Hello Carl');
  });" >> src/__tests__/Greeter.test.ts
```

### 實驗
編譯
將會產生lib資料夾，裡面會有 index.js 與 index.d.ts
這兩個檔案會供人使用
```bash=
npm run build
```

測試
它會找所有 src/__tests__ 資料夾下
檔名符合 xxxx.test.ts 或 xxxx.spec.ts 
(這要看 jestconfig.json 的設定)
```bash=
npm run test
```

格式測試
```bash=
npm run format
npm run lint
```

## 推送
### 操作
將 git 的變動 commit
第一次使用，要設定 git 的 user.name 與 user.email 
否則無法直接用 vscode 處理 commit 與 上傳
```bash=
# 設定 (不能空白，會斷開)
git config user.name Emmanuel-Lo
git config user.email snowray712000@gmail.com
# 看 config 檔
cat .git/config
```

登入 npm
```bash=
npm login
# 然後輸入 username, password, email
npm publish
```

新版本
當有新版本的時候，可用下指令
原本是 1.0.0 會變 1.0.1 
```bash=
npm version patch
npm publish 
```
## {另一個專案}使用

打開另一個專案，使用
就發現，可以成功使用了
```bash=
npm install 專案名稱 
```

