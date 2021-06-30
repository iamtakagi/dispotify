# dispotify
🎧 Spotify to Discord Notifier Web アプリ 

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/iamtakagi/dispotify)](https://github.com/iamtakagi/dispotify/releases)
[![Deploy](https://github.com/iamtakagi/dispotify/actions/workflows/deploy.yml/badge.svg)](https://github.com/iamtakagi/dispotify/actions/workflows/deploy.yml)
[![license](https://img.shields.io/github/license/iamtakagi/dispotify)](https://github.com/iamtakagi/dispotify/blob/master/LICENSE)
[![issues](https://img.shields.io/github/issues/iamtakagi/dispotify)](https://github.com/iamtakagi/dispotify/issues)
[![pull requests](https://img.shields.io/github/issues-pr/iamtakagi/dispotify)](https://github.com/iamtakagi/dispotify/pulls)

## 特徴
- Spotify x Discord
- 場所を選ばず、手軽にアクセス可能な、Webアプリ
- ユーザーは設定不要
- 誰でも、簡単なUIで設定可能
- Dockerで簡単にセットアップ可能

## 概要・開発動機
- 以前から公式のSpotifyアプリに再生履歴が無いことに不便さを感じており、開発しました。
    - このアプリを利用するメリット
        - Spotifyの再生履歴がわかる
        - Discordで共有できる
        - Discord Presence だと見逃しがちな他ユーザーの聴いた曲がわかる。(チャンネル非公開でも使えるので、プライベート利用もOK)
        - 等の点において便利な活用ができると考えました。
- 素人が作ったWebアプリなので、不具合等、至らない点あると思いますが、改善や新機能を実装する開発を続けていくと思います。
- 貢献について
    - 新機能提案・不具合報告は [Issues](https://github.com/iamtakagi/dispotify/issues) にお願いします。
    - [Pull Requests](https://github.com/dispotify/pulls) も歓迎しています。

## 使い方
- [公開版](https://dispotify.iamtakagi.net) もありますが、ユーザー各自でインストールすることを推奨しています。
  - 多数ユーザーで利用されてしまうと、安定した動作が期待できません。そのため、できるだけ各自でインストールしてください。24時間継続的に動かせる環境がない等の事情がある方は [そちら](https://dispotify.iamtakagi.net) を利用してください。

## インストール
```console
touch docker-compose.yml
# Database Url 設定
touch backend/prisma/.env
docker-compose up -d --build
```

`docker-compose.yml`
```yml
version: '3.8'

services:
  nginx:
    build: nginx
    ports:
        - 80:80
    environment:
        TZ: Asia/Tokyo
    restart: always

  postgres:
    container_name: postgres
    image: postgres
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: dispotify
    restart: always
    volumes:
      - ./db:/docker-entrypoint-initdb.d
      - postgres-data:/var/lib/postgresql/data
    networks:
      - postgres-network
    ports:
      - 5432:5432

  backend:
    container_name: backend
    image: iamtakagi/dispotify_backend:latest
    environment:
      TZ: Asia/Tokyo
      HOST: 0.0.0.0
      PORT: 3001
      BASE_URI: /api
      FRONTEND_BASE_URI: https://dispotify.iamtakagi.net
      SPOTIFY_REDIRECT_URI: https://dispotify.iamtakagi.net/api/auth/callback
      # 入力必須
      SPOTIFY_CLIENT_ID: xxx
      SPOTIFY_CLIENT_SECRET: xxx
      # 再生中の音楽を検知するタスクのインターバル: 25秒以上を推奨 (それ以下にすると SpotifyAPI のRateLimit に引っ掛かり Too Many Request 429 の応答エラーが発生する可能性有)
      PLAYER_WATCH_INTERVAL: 25000
    restart: always
    networks:
      - postgres-network
    ports:
      - 3001:3001
    links: 
      - postgres
    depends_on:
      - postgres

  frontend:
    container_name: frontend
    image: iamtakagi/dispotify_frontend:latest
    environment:
      TZ: Asia/Tokyo
      HOST: 0.0.0.0
      PORT: 3000
      BACKEND_BASE_URI: https://dispotify.iamtakagi.net/api/
    restart: always
    ports:
      - 3000:3000
    links: 
      - backend
    depends_on: 
      - backend

volumes:
    postgres-data:
      driver: local

networks:
    postgres-network:
      driver: bridge
```

## 開発
```console
cd backend && yarn
cd frontend && yarn && yarn prisma:generate
```

`docker-compose.yml`
```yml
version: '3.8'

services:
  nginx:
    build: nginx
    ports:
        - 80:80
    environment:
        TZ: Asia/Tokyo
    restart: always

  postgres:
    container_name: postgres
    image: postgres
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: dispotify
    restart: always
    volumes:
      - ./db:/docker-entrypoint-initdb.d
      - postgres-data:/var/lib/postgresql/data
    networks:
      - postgres-network
    ports:
      - 5432:5432

  backend:
    container_name: backend
    build: backend
    environment:
      TZ: Asia/Tokyo
      HOST: 0.0.0.0
      PORT: 3001
      BASE_URI: /api
      FRONTEND_BASE_URI: http://localhost:3000
      SPOTIFY_REDIRECT_URI: http://localhost:3001/api/auth/callback
      # 入力必須
      SPOTIFY_CLIENT_ID: xxx
      SPOTIFY_CLIENT_SECRET: xxx
      # 再生中の音楽を検知するタスクのインターバル: 25秒以上を推奨 (それ以下にすると SpotifyAPI のRateLimit に引っ掛かり Too Many Request 429 の応答エラーが発生する可能性有)
      PLAYER_WATCH_INTERVAL: 25000
    restart: always
    networks:
      - postgres-network
    ports:
      - 3001:3001
    links:
      - postgres
    depends_on:
      - postgres

  frontend:
    container_name: frontend
    build: frontend
    environment:
      TZ: Asia/Tokyo
      HOST: 0.0.0.0
      PORT: 3000
      BACKEND_BASE_URI: http://localhost:3001/api
    restart: always
    ports:
      - 3000:3000
    links: 
      - backend
    depends_on: 
      - backend

volumes:
    postgres-data:
      driver: local

networks:
    postgres-network:
      driver: bridge
```