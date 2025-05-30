ARG NODE_VERSION=22.16.0

FROM node:${NODE_VERSION}-alpine AS build

WORKDIR /app

COPY --link package*.json ./

RUN npm install

RUN npm install -g @angular/cli@19

COPY --link . .

RUN npm run build --configuration=production

FROM nginx:alpine

COPY --link --from=build /app/dist/angular/browser /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]