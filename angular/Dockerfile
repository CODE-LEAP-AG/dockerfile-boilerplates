ARG NODE_VERSION=22.16.0

FROM node:${NODE_VERSION} AS build

WORKDIR /app

COPY --link package*.json ./

RUN npm install

RUN npm install -g @angular/cli@19

COPY --link . .

RUN ng build --configuration=production

FROM nginx:latest

COPY --link --from=build /app/dist/angular/browser /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]