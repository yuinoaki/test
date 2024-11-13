pipeline {
    agent any // エージェントをanyに変更
    stages {
        stage('Build') {
            steps {
                script {
                    // txtファイルを作成
                    writeFile file: 'build.txt', text: 'Build stage output'
                    // 成果物として保存（stashで保存）
                    stash includes: 'build.txt', name: 'build-txt'
                }
            }
        }
        stage('Static Analysis') {
            steps {
                script {
                    // txtファイルを作成
                    writeFile file: 'static_analysis.txt', text: 'Static analysis output'
                    // 成果物として保存（stashで保存）
                    stash includes: 'static_analysis.txt', name: 'static-analysis-txt'
                }
            }
        }
        stage('Unit Test') {
            steps {
                script {
                    // txtファイルを作成
                    writeFile file: 'unit_test.txt', text: 'Unit test output'
                    // 成果物として保存（stashで保存）
                    stash includes: 'unit_test.txt', name: 'unit-test-txt'
                }
            }
        }
        stage('Save to SharePoint') {
            steps {
                script {
                    // フォルダ名を作成するための変数を取得
                    def branchName = env.BRANCH_NAME ?: 'main' // ブランチ名がない場合のデフォルト値
                    def buildNumber = env.BUILD_NUMBER
                    // コミットハッシュの取得（先頭7桁）
                    def commitHashOutput = bat(returnStdout: true, script: '@echo off & git rev-parse HEAD')
                    def commitHashLines = commitHashOutput.readLines()
                    def commitHash = commitHashLines[commitHashLines.size()-1].trim().substring(0,7)
                    // 日付を取得
                    def date = new Date().format('yyyyMMdd')
                    // フォルダ名を作成
                    def folderName = "${branchName}_${buildNumber}_${commitHash}_${date}"
                    // フォルダ構造を作成
                    bat "mkdir \"${folderName}\""
                    bat "mkdir \"${folderName}\\Build\""
                    bat "mkdir \"${folderName}\\Static Analysis\""
                    bat "mkdir \"${folderName}\\Unit Test\""
                    // 各フォルダに関連付けられたファイルをunstash
                    dir("${folderName}\\Build") {
                        unstash 'build-txt'
                    }
                    dir("${folderName}\\Static Analysis") {
                        unstash 'static-analysis-txt'
                    }
                    dir("${folderName}\\Unit Test") {
                        unstash 'unit-test-txt'
                    }
                    // コンソールログを取得し、保存
                    def jobName = env.JOB_NAME
                    // ファイル名に使用できない文字を置換（スラッシュをアンダースコアに置換）
                    def sanitizedJobName = jobName.replaceAll('[\\\\/:*?"<>|]', '_')
                    def jenkinsUrl = env.JENKINS_URL
                    // ジョブのフルパスを取得（スラッシュを置き換え）
                    def jobFullPath = jobName.replaceAll('/', '/job/')
                    // コンソールログのURLを作成
                    def consoleTextUrl = "${jenkinsUrl}job/${jobFullPath}/${buildNumber}/consoleText"
                    // コンソールログを取得
                    bat "curl \"${consoleTextUrl}\" --output \"${sanitizedJobName}_${buildNumber}.txt\" -k"
                    // コンソールログをフォルダに移動
                    bat "move \"${sanitizedJobName}_${buildNumber}.txt\" \"${folderName}\""
                    // 最終的にzip化する
                    bat "powershell Compress-Archive -Path \"${folderName}\" -DestinationPath \"${folderName}.zip\""
                    // zipファイルを成果物として保存
                    archiveArtifacts artifacts: "${folderName}.zip"
                }
            }
        }
    }
}
