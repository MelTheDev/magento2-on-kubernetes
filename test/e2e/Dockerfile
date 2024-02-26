FROM cypress/base:20.11.0@sha256:3057ac15e8a258e4c7cb3e670d029037d7222e80440704ee3c0fa451c8a67cec

WORKDIR /e2e
COPY package.json /e2e/package.json
COPY package-lock.json /e2e/package-lock.json

RUN npm install

COPY . /e2e
