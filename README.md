# Jenkins Study

JenkinsをDocker上に構築し、Freestyle Job、Pipeline、Jenkinsfile、GitHub連携、SCM Polling、Next.jsのCIまでを実践した学習リポジトリです。

最終的に以下の流れを構築しました。

```text
VS Code
  ↓
git commit / git push
  ↓
GitHub
  ↓
Jenkins SCM Polling
  ↓
変更を検知
  ↓
Jenkinsfile取得
  ↓
npm ci
  ↓
npm run lint
  ↓
npm run build
  ↓
SUCCESS / FAILURE
```

---

# 1. 学習目的

これまで以下のCI/CDを学習しました。

- GitHub Actions
- GitLab CI/CD

今回はJenkinsを利用し、CI/CDサービスを利用するだけではなく、

- Jenkins Serverの構築
- Jobの作成
- Pipeline
- Jenkinsfile
- GitHub連携
- Build Trigger
- Build環境
- CIの失敗と復旧

までを実践することを目的としました。

---

# 2. 学習環境

```text
Windows 11
Docker Desktop
Jenkins
GitHub
Next.js
Node.js
npm
VS Code
```

Jenkins自体はDockerコンテナとして起動しました。

```text
Windows 11
   ↓
Docker Desktop
   ↓
Jenkins Container
   ↓
http://localhost:8080
```

---

# 3. Jenkinsとは

Jenkinsはオープンソースの自動化サーバーです。

CI/CD Pipelineを構築でき、

```text
Source Code
   ↓
Build
   ↓
Test
   ↓
Deploy
```

などの処理を自動化できます。

GitHub ActionsやGitLab CI/CDとの大きな違いとして、JenkinsではCI/CDを実行する基盤自体を自分で構築・管理する構成を取ることができます。

---

# 4. JenkinsをDockerで構築

まずJenkinsのデータを永続化するため、Docker Volumeを作成しました。

```powershell
docker volume create jenkins_home
```

Jenkinsを起動しました。

```powershell
docker run -d --name jenkins `
  -p 8080:8080 `
  -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  jenkins/jenkins:lts-jdk21
```

確認：

```powershell
docker ps
```

ブラウザから以下へアクセスしました。

```text
http://localhost:8080
```

初期設定後、Jenkins Dashboardへログインできることを確認しました。

---

# 5. Docker VolumeによるJenkinsデータの永続化

今回、

```text
jenkins_home
```

というDocker Volumeを使用しました。

```text
Jenkins Container
      ↓
jenkins_home
      ↓
/var/jenkins_home
```

Jenkinsのコンテナそのものを削除しても、Volumeを削除しなければJenkinsの設定やJobなどのデータを再利用できます。

後半でJenkinsコンテナを作り直した際にも、同じVolumeをマウントすることで設定を引き継げることを確認しました。

---

# 6. Freestyle Job

最初にJenkinsの基本を理解するため、Freestyle Jobを作成しました。

Job名：

```text
hello-jenkins
```

Build Stepとして、

```bash
echo "Hello Jenkins"
```

を設定しました。

実行結果：

```text
Hello Jenkins
Finished: SUCCESS
```

これにより、

```text
Job
 ↓
Build
 ↓
Shell Command
 ↓
Console Output
```

というJenkinsの基本的な実行単位を確認しました。

---

# 7. Pipeline

次にPipeline Jobを作成しました。

Job名：

```text
hello-pipeline
```

最初はJenkinsのGUI上に直接Pipeline Scriptを記述しました。

```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello Jenkins Pipeline'
            }
        }
    }
}
```

実行結果：

```text
Hello Jenkins Pipeline
Finished: SUCCESS
```

---

# 8. FreestyleとPipelineの違い

今回の学習では、以下の違いを確認しました。

## Freestyle

GUIから処理を設定します。

```text
Jenkins GUI
   ↓
Build Step
   ↓
Shell
   ↓
echo
```

## Pipeline

CI/CD処理をコードとして定義できます。

```text
Pipeline
   ↓
Stage
   ↓
Steps
```

Pipelineをコード化することで、CI/CDの処理自体をGitで管理できます。

---

# 9. Jenkinsfile

GUIへ直接Pipelineを書くのではなく、

```text
Jenkinsfile
```

を作成しました。

最初のJenkinsfile：

```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello Jenkins Pipeline'
            }
        }
    }
}
```

これをGitHubへpushしました。

```text
GitHub
└── Jenkinsfile
```

---

# 10. GitHubとJenkinsを連携

JenkinsのPipeline設定を、

```text
Pipeline script
```

から、

```text
Pipeline script from SCM
```

へ変更しました。

設定：

```text
SCM
Git

Repository URL
https://github.com/RH-devop/jenkins-study.git

Branch
*/main

Script Path
Jenkinsfile
```

Public Repositoryのため、今回Credentialsは設定していません。

実行すると、

```text
Obtained Jenkinsfile from git
```

が表示され、GitHub上のJenkinsfileを取得できることを確認しました。

構成：

```text
GitHub
  ↓
Jenkins
  ↓
Jenkinsfile取得
  ↓
Pipeline実行
```

---

# 11. 複数StageのPipeline

次にPipelineを、

```text
Build
 ↓
Test
 ↓
Deploy
```

の3Stageへ変更しました。

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
            }
        }
    }
}
```

これにより、Pipelineが複数のStageを順番に実行することを確認しました。

---

# 12. SCM Pollingによる自動実行

当初は、

```text
git push
 ↓
GitHub
 ↓
Jenkinsで手動ビルド
```

となっていました。

CI/CDとして自動化するため、JenkinsのBuild Triggerで、

```text
SCMをポーリング
```

を設定しました。

スケジュール：

```text
H/2 * * * *
```

学習環境では約2分間隔でGitHubの変更を確認する構成としました。

変更がある場合：

```text
Jenkins
  ↓
GitHubを定期確認
  ↓
変更検知
  ↓
Pipeline自動起動
```

Polling Logでは、

```text
Changes found
```

を確認しました。

また、自動起動したBuildでは、

```text
Started by an SCM change
```

が表示されました。

---

# 13. SCM PollingとWebhook

今回のJenkinsは、

```text
http://localhost:8080
```

で動作しています。

そのため、インターネット上のGitHubからローカルPCのJenkinsへ直接Webhookを送信する構成にはせず、SCM Pollingを使用しました。

SCM Polling：

```text
Jenkins
  ↓
GitHub変わった？
  ↓
GitHub変わった？
  ↓
GitHub変わった？
```

Webhook：

```text
git push
  ↓
GitHub
  ↓
Webhook
  ↓
Jenkins
  ↓
Pipeline
```

今回のようなローカル学習環境ではSCM Pollingが利用しやすい一方、実環境では即時性や効率の観点からWebhookなどのイベント駆動方式を利用する構成も重要だと理解しました。

---

# 14. Node.js / npmが存在しない問題

Next.jsをJenkinsでBuildしようとした際、Jenkinsコンテナ内にNode.jsとnpmが存在するか確認しました。

```powershell
docker exec jenkins node --version
docker exec jenkins npm --version
```

結果：

```text
exec: "node": executable file not found in $PATH
exec: "npm": executable file not found in $PATH
```

使用していたJenkins Imageには、今回のNext.js Buildに必要なNode.js / npmが入っていませんでした。

つまり、

```text
Jenkinsが動く環境
```

と、

```text
アプリケーションをBuildできる環境
```

は別であり、Pipelineで使用するツールを実行環境側に用意する必要があります。

---

# 15. Node.js入りJenkins Imageを作成

Dockerfileを作成しました。

```dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

RUN apt-get update \
    && apt-get install -y nodejs npm \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Build：

```powershell
docker build -t jenkins-node .
```

これにより、

```text
jenkins-node
├── Jenkins
├── Java
├── Node.js
└── npm
```

という独自Imageを作成しました。

---

# 16. Jenkinsコンテナを再作成

既存Jenkinsを停止・削除しました。

```powershell
docker stop jenkins
docker rm jenkins
```

新しいImageから起動しました。

```powershell
docker run -d --name jenkins `
  -p 8080:8080 `
  -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  jenkins-node
```

同じ `jenkins_home` Volumeを使用したため、JenkinsのJobや設定を引き継ぐことができました。

確認：

```powershell
docker exec jenkins node --version
docker exec jenkins npm --version
```

結果：

```text
v20.19.2
9.2.0
```

---

# 17. Next.jsアプリ作成

Jenkinsで実際のCIを行うため、Next.jsアプリを作成しました。

```powershell
npx.cmd create-next-app@latest app
```

構成：

```text
jenkins-study/
├── Jenkinsfile
├── Dockerfile
└── app/
    ├── package.json
    ├── package-lock.json
    ├── app/
    └── ...
```

---

# 18. Next.js CI Pipeline

Jenkinsfileを以下の構成へ変更しました。

```groovy
pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                dir('app') {
                    sh 'npm ci'
                }
            }
        }

        stage('Lint') {
            steps {
                dir('app') {
                    sh 'npm run lint'
                }
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'npm run build'
                }
            }
        }
    }
}
```

処理：

```text
GitHub
  ↓
Jenkins
  ↓
Jenkinsfile
  ↓
Install
npm ci
  ↓
Lint
npm run lint
  ↓
Build
npm run build
  ↓
SUCCESS
```

実際に、

```text
Compiled successfully
Finished: SUCCESS
```

まで確認できました。

---

# 19. SCM Pollingのトラブルシュート

学習中、Jenkinsが自動起動しないように見える事象が発生しました。

Polling Logでは、

```text
Last Built Revision
Latest remote head revision
No changes
```

などの情報を確認できます。

GitHub側のRevision確認：

```powershell
docker exec jenkins git ls-remote https://github.com/RH-devop/jenkins-study.git refs/heads/main
```

Workspace側のRevision確認：

```powershell
docker exec jenkins sh -c "cd /var/jenkins_home/workspace/hello-pipeline && git log -1 --oneline"
```

これにより、

- GitHub上の最新Revision
- Jenkinsが認識しているLast Built Revision
- WorkspaceにCheckoutされているRevision

は分けて確認する必要があることを学びました。

コンテナ再作成後、一時的にSCMの基準状態とWorkspaceの状態に差が見られました。

一度手動ビルドを実行して最新RevisionをCheckoutした後、以降のSCM Pollingでは正常に変更を検知して自動実行できることを確認しました。

ただし、この事象についてはコンテナ再作成が直接的な原因だったとは断定していません。

---

# 20. コンテナ運用とWorkspaceについての所感

Jenkins Controllerをコンテナ化する場合、コンテナ自体は再作成可能です。

```text
Jenkins Container
      ↓
削除 / 再作成
      ↓
Persistent Volume
      ↓
Jenkinsの永続データを保持
```

一方、Build時のWorkspaceについては、永続的に存在することを前提にしない設計が重要だと感じました。

実際のJenkinsでは、

```text
Jenkins Controller
       ↓
     Agent
       ↓
   Workspace
       ↓
    Build
```

のようにControllerとBuild実行環境を分離できます。

Kubernetes Agentなどでは、

```text
Pipeline開始
  ↓
Agent Pod作成
  ↓
Git Checkout
  ↓
Build / Test
  ↓
Pipeline終了
  ↓
Agent Pod削除
```

のような一時的なBuild環境も利用できます。

今回の学習から、Jenkins本体の永続データとBuild Workspaceは分けて考える必要があると理解しました。

---

# 21. Pipelineを意図的に失敗させる

CIの失敗時の動作を確認するため、Lint Stageを一時的に変更しました。

```groovy
stage('Lint') {
    steps {
        dir('app') {
            sh 'exit 1'
        }
    }
}
```

結果：

```text
Install
  ↓
SUCCESS

Lint
  ↓
exit 1
  ↓
FAILURE

Build
  ↓
Skipped
```

Console Output：

```text
Stage "Build" skipped due to earlier failure(s)
ERROR: script returned exit code 1
Finished: FAILURE
```

これにより、前段Stageが失敗すると後続Stageが実行されないことを確認しました。

---

# 22. Pipelineの修正と復旧

Lint Stageを元に戻しました。

```groovy
stage('Lint') {
    steps {
        dir('app') {
            sh 'npm run lint'
        }
    }
}
```

GitHubへpushすると、SCM Pollingによって自動起動しました。

```text
Started by an SCM change
```

結果：

```text
Install
  ↓
SUCCESS

Lint
  ↓
SUCCESS

Build
  ↓
SUCCESS

Finished: SUCCESS
```

これにより、

```text
Pipeline失敗
  ↓
Console Output確認
  ↓
原因特定
  ↓
コード修正
  ↓
git push
  ↓
SCM Polling
  ↓
自動再実行
  ↓
SUCCESS
```

というCIの基本的なトラブルシュートを体験しました。

---

# 23. GitHub Actions / GitLab CI/CD / Jenkins 比較

| 項目 | GitHub Actions | GitLab CI/CD | Jenkins |
|---|---|---|---|
| Pipeline定義 | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `Jenkinsfile` |
| 主な記述形式 | YAML | YAML | Groovy / Declarative Pipeline |
| 実行環境 | Runner | Runner | Controller / Agent |
| Git連携 | GitHubと統合 | GitLabと統合 | GitHub / GitLab等と連携 |
| 自動起動 | GitHub Event | GitLab Event | Webhook / SCM Polling等 |
| 基盤管理 | サービス側に任せやすい | サービス側に任せやすい | 自前管理可能 |
| カスタマイズ性 | 高い | 高い | 非常に高い |
| 運用負荷 | 比較的小さい | 比較的小さい | 比較的大きい |

---

# 24. 3つを実際に触った所感

GitHub ActionsとGitLab CI/CDは、

```text
Git Repository
     +
CI/CD Service
```

が最初から強く統合されているため、比較的簡単にCI/CDを開始できました。

一方Jenkinsでは、

```text
Jenkins Server
     ↓
Build Environment
     ↓
Git
     ↓
Trigger
     ↓
Pipeline
```

まで自分で考える必要がありました。

そのため設定や運用の手間は増えますが、CI/CD基盤そのものを構築している感覚が強く、自由度の高さを感じました。

特に今回、Jenkins公式ImageにNode.jsが存在しなかったことで、

```text
Jenkinsが動く環境
```

と、

```text
アプリケーションをBuildできる環境
```

は同じではないことを実際に体験できました。

---

# 25. SCM Pollingについての所感

今回のローカル環境では、

```text
Jenkins = localhost:8080
```

だったため、SCM Pollingを使用しました。

学習用途では、

```text
GitHub変更
  ↓
Jenkinsが検知
  ↓
Pipeline自動実行
```

というTriggerの仕組みを理解するのに有効でした。

一方、PollingではJenkins側から定期的にGitHubへ問い合わせる必要があります。

そのため実環境では、環境やネットワーク構成に応じてWebhookなどのイベント駆動方式を利用する方が適しているケースが多いと感じました。

今回のSCM Pollingは、

> ローカルJenkinsでCI/CDの自動起動を学習するための手段

として利用しました。

---

# 26. GitHub Actionsとの対応イメージ

完全な1対1対応ではありませんが、今回の学習では以下のように整理しました。

```text
GitHub Actions                Jenkins

Workflow                      Pipeline
   ↓                             ↓
Job                           Stage
   ↓                             ↓
Step                          Step
```

ファイルとしては、

```text
GitHub Actions
.github/workflows/ci.yml

GitLab CI/CD
.gitlab-ci.yml

Jenkins
Jenkinsfile
```

という違いがあります。

---

# 27. 今回理解したJenkinsの構成

```text
Developer
   ↓
git push
   ↓
GitHub
   ↓
SCM Polling
   ↓
Jenkins Controller
   ↓
Jenkinsfile
   ↓
Pipeline
   ├── Install
   ├── Lint
   └── Build
```

さらに実環境では、

```text
Jenkins Controller
       ↓
     Agent
       ↓
Build Environment
```

のように、ControllerとBuild実行環境を分離できることも理解しました。

---

# 28. 学習成果

今回の学習で以下を実践しました。

- DockerによるJenkins構築
- Docker VolumeによるJenkinsデータ永続化
- Jenkins初期設定
- Freestyle Job
- Console Output
- Pipeline Job
- Declarative Pipeline
- Jenkinsfile
- Pipeline as Code
- GitHub連携
- Pipeline script from SCM
- 複数Stage
- SCM Polling
- Git変更によるPipeline自動起動
- DockerfileによるJenkins Image拡張
- Node.js / npmのBuild環境構築
- Next.jsアプリ作成
- `npm ci`
- `npm run lint`
- `npm run build`
- Pipelineの意図的な失敗
- 後続StageのSkip
- Console Outputによる原因確認
- Pipeline修正
- 自動再実行
- Jenkins Controller / Agentの基本概念
- Workspaceと永続データの違い
- GitHub Actions / GitLab CI/CD / Jenkinsの比較

---

# 29. 最終構成

```text
Windows 11
│
├── VS Code
│     │
│     └── jenkins-study
│          ├── README.md
│          ├── Jenkinsfile
│          ├── Dockerfile
│          └── app/
│
├── Git
│     ↓
│   GitHub
│     ↓
│
└── Docker Desktop
      │
      └── Jenkins Container
            │
            ├── Jenkins
            ├── Java
            ├── Node.js
            ├── npm
            │
            ├── SCM Polling
            │      ↓
            │    GitHub
            │
            └── Pipeline
                   │
                   ├── npm ci
                   ├── npm run lint
                   └── npm run build
```

---

# 30. Phase 8 完了

Phase 8では、Jenkinsを単に操作するだけではなく、

```text
CI/CD基盤を構築
   ↓
Pipelineをコード化
   ↓
GitHubと連携
   ↓
変更を自動検知
   ↓
実際のアプリをCI
   ↓
失敗
   ↓
ログから原因確認
   ↓
修正
   ↓
自動復旧
```

までを一通り実践しました。

GitHub Actions / GitLab CI/CDでは意識しにくかった、

- CI/CD Server
- Build環境
- 永続化
- Trigger
- Workspace
- Controller / Agent

といったCI/CD基盤側の仕組みを、Jenkinsを通して理解することができました。