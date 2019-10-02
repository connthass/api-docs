FROM node:latest

WORKDIR /swagger

COPY package*.json ./
COPY . .

RUN npm install

CMD [ "npm", "run", "watch" ]