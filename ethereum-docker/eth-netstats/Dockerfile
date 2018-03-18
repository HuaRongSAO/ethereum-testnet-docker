FROM node:8-alpine

RUN apk add --update git

RUN git clone https://github.com/cubedro/eth-netstats --depth 1

WORKDIR /eth-netstats

RUN npm install -g cnpm --registry=https://registry.npm.taobao.org
RUN cnpm install
RUN cnpm install -g grunt-cli
RUN grunt

EXPOSE 3000

CMD ["npm", "start"]
