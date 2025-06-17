# Jenkins

## What is jenkins?

- Open source 
- continuous integrations and continuous development (CI/CD) tools

## Use cases
1. Continuous Integration
- automate test, lint, 測試覆蓋率

2. Continuous Development
- automate deployment to test, staging, production environment

3. Docker
- automate creation 
- automate deploy image

4. Terraform/ Ansible
- 部署基礎設施或機器設定

5. 測試報告
- 整合JUnit, Allure,...

6. DevOps 流程自動化
- 持續交付，版本發布，監控觸發等

## Jenkins的定位
自動化engine platform

### core functinos
- 將developer重複做的開發測試部署流程，自動化，流程化，版本化

### What Jenkins can do?
- 在push code的時候自動完成test跟build
- 發布Docker Image, deploy container application
- 建立雲端資源，初始化環境
- 和團隊share可以reuse的deploy流程


## Stage 1: CI basics & Jenkins Pipeline 建立
### 目的
完成一個Jenkins Pipeline Job，可以自動拉repo，執行測試與建構

### 準備環境
- Jenkins 安裝
- 一個Github專案
- plugins：
    - Git Plugin
    - Pipeline Plugin

### 範例1: Nodejs
1. 建立Jenkins job (Pipeline 類型)
2. 撰寫Jenkinsfile (基本)
3. 執行N `npm install/test/build`
4. 將結果輸出為console log

```
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/your-org/your-project.git'
      }
    }
    stage('Install') {
      steps {
        sh 'npm install'       //sh 代表的是步驟
      }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }
  }
}

```

### 範例2: Nodejs完整的
```
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { git 'https://github.com/your-org/project.git' }
    }
    stage('Install') {
      steps { sh 'npm install' }
    }
    stage('Test') {
      steps { sh 'npm test' }
    }
    stage('Build') {
      steps { sh 'npm run build' }
    }
    stage('Deploy') {
      steps {
        sh 'scp -r ./dist user@server:/app'
        sh 'ssh user@server "pm2 restart app"'
      }
    }
  }
  post {
    success { echo 'All done!' }
    failure { echo 'Something went wrong!' }
  }
}

```

## Stages 說明
這邊的Jenkins的pipeline用上的stages說明
1. stage('Checkout')
用處： 拉code
這個就是ci cd pipeline的entrypoint, 負責從git裡面吧code抓下來
```
stage('Checkout') {
  steps {
    git 'https://github.com/your-org/your-project.git'
  }
}
```
首先當然是從提供的git去拉source code，當然它可以處理multi-branch, submodule, and GitHub credentials.

2. stage('Install')
用處： 安裝project的dependency
一般來說想node，dependency就是`npm install`
python的dependency就是`pip install -r requirements.txt`

```
stage('Install') {
  steps {
    sh 'npm install'
  }
}
```

3. stage('Test')
用途：自動化測試
這邊就是去自動執行單元測試，整合測試，讓你確認push會不會破壞其他的功能。
```
stage('Test') {
  steps {
    sh 'npm test'
  }
}
```
這邊也可以整合其他auto testing 的工具：
- JUnit
- Allure
- Coverage Report

4. stage('lint') 
用途：靜態program code analysis
這一階段適用於分析program code 的風格錯誤，潛在錯誤
例如：eslint, flake8
```
stage('Lint') {
  steps {
    sh 'npm run lint'
  }
}
```

5. stage('Build')
用途：建構application的產物
這邊就會把program source code轉化成可以deploy的version

- Node.js => npm run build
- React => vite build
- Java => mvn package
- Rust => cargo build

6. stage('Docker Build')
用途： 建立Docker image
這邊就是封裝，把application裝起來變成container
```
stage('Docker Build') {
  steps {
    sh 'docker build -t your-image-name .'
  }
}
```

7. stage('Deploy')
用處： deploy到test/staging/production的環境
這個階段一般就是負責：
- 複製build的產物到remote server
- 通過ssh 來下指令
- 呼叫helm/k8s/ansible/terraform的指令
```
stage('Deploy') {
  steps {
    sh 'scp -r ./dist user@vm:/var/www/app'
    sh 'ssh user@vm "sudo systemctl restart my-app"'
  }
}
```

8. post block
這邊其實不是stage，這個是pipeline結束之後的動作
```
post {
  success {
    echo "Success!"
  }
  failure {
    echo "Failed."
  }
  always {
    cleanWs()
  }
}
```
在這個post的區塊可以做到的就是
- 發通知
- 清楚工作目錄
- 傳送報表


## 什麼是sh？
sh = 一個step的指令，這個是用在Jenkins agent 的shell（一般是bash）中執行命令。

```
sh '指令內容'
```
- 其實就跟Linux command一樣的
- 然後也只適用於linux 跟 MacOS