# Deploy

Similar to [GitBook](https://www.gitbook.com), you can deploy files to GitHub Pages, GitLab Pages or VPS.

## Backup cPanel/WHM ke Object Storage

!> Sebelum melakukan aktifitas ini pastikan telah melakukan pembuatan Bucket terlebih dahulu

1. Login kedalam WHM lalu menuju ke Menu :

Backup > Backup Configuration

2. Selanjutnya masuk kedalam tab 

Additional Destination > Destination Type > Pilih S3 Compatible > Create a New Destination

3. Dapat mengisikan konfigurasi seperti berikut : 

Destination Name : <Dapat diisikan nama backup> 

Folder : <Dapat diisikan nama folder yang berada dalam bucket Arupa Object Storage>  

S3 Endpoint : <Dapat diisi hostname endpoint Arupa Object Storage> 

Bucket : <Dapat diisikan bucket yang sebelum telah terdapat pada Object Storage> 

Access Key ID : <Dapat diisikan Access Key yang diberikan> 

Secret Access Key :  < Dapat diisikan Secret Access Key yang diberikan> 

4. Silahkan klik "Save Destination" setelah itu dapat mengklik tombol "Validate"

5. Setelah melakukan proses validasi akan muncul simbol "Checked" pada konfigurasi backup


## GitLab Pages

If you are deploying your master branch, include `.gitlab-ci.yml` with the following script:

?> The `.public` workaround is so `cp` doesn't also copy `public/` to itself in an infinite loop.

```YAML
pages:
  stage: deploy
  script:
  - mkdir .public
  - cp -r * .public
  - mv .public public
  artifacts:
    paths:
    - public
  only:
  - master
```

!> You can replace script with `- cp -r docs/. public`, if `./docs` is your Docsify subfolder.

## Firebase Hosting

!> You'll need to install the Firebase CLI using `npm i -g firebase-tools` after signing into the [Firebase Console](https://console.firebase.google.com) using a Google Account.

Using Terminal determine and navigate to the directory for your Firebase Project - this could be `~/Projects/Docs` etc. From there, run `firebase init`, choosing `Hosting` from the menu (use **space** to select, **arrow keys** to change options and **enter** to confirm). Follow the setup instructions.

You should have your `firebase.json` file looking similar to this (I changed the deployment directory from `public` to `site`):

```json
{
  "hosting": {
    "public": "site",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"]
  }
}
```

Once finished, build the starting template by running `docsify init ./site` (replacing site with the deployment directory you determined when running `firebase init` - public by default). Add/edit the documentation, then run `firebase deploy` from the base project directory.

## VPS

Try following nginx config.

```nginx
server {
  listen 80;
  server_name  your.domain.com;

  location / {
    alias /path/to/dir/of/docs/;
    index index.html;
  }
}
```

## Netlify

1.  Login to your [Netlify](https://www.netlify.com/) account.
2.  In the [dashboard](https://app.netlify.com/) page, click **New site from Git**.
3.  Choose a repository where you store your docs, leave the **Build Command** area blank, fill in the Publish directory area with the directory of your `index.html`, for example it should be docs if you populated it at `docs/index.html`.

### HTML5 router

When using the HTML5 router, you need to set up redirect rules that redirect all requests to your `index.html`, it's pretty simple when you're using Netlify, create a file named `_redirects` in the docs directory, add this snippet to the file and you're all set:

```sh
/*    /index.html   200
```

## ZEIT Now

1. Install [Now CLI](https://zeit.co/download), `npm i -g now`
2. Change directory to your docsify website, for example `cd docs`
3. Deploy with a single command, `now` 

## AWS Amplify

1. Set the routerMode in the Docsify project `index.html` to *history* mode.

```html
<script>
    window.$docsify = {
      loadSidebar: true,
      routerMode: 'history'
    }
</script>
```

2. Login to your [AWS Console](https://aws.amazon.com).
3. Go to the [AWS Amplify Dashboard](https://aws.amazon.com/amplify).
4. Choose the **Deploy** route to setup your project.
5. When prompted, keep the build settings empty if you're serving your docs within the root directory. If you're serving your docs from a different directory, customise your amplify.yml

```yml
version: 0.1
frontend:
  phases:
    build:
      commands: 
        - echo "Nothing to build"
  artifacts:
    baseDirectory: /docs
    files:
      - '**/*'
  cache:
    paths: []

```

6. Add the following Redirect rules in their displayed order. Note that the second record is a PNG image where you can change it with any image format you are using. 

| Source address | Target address | Type          |
|----------------|----------------|---------------|
| /<*>.md        | /<*>.md        | 200 (Rewrite) |
| /<*>.png       | /<*>.png       | 200 (Rewrite) |
| /<*>           | /index.html    | 200 (Rewrite) |        


## Docker

- Create docsify files 

  You need prepare the initial files instead of making in container.  
  See the [Quickstart](https://docsify.js.org/#/quickstart) section for instructions on how to create these files manually or using [docsify-cli](https://github.com/docsifyjs/docsify-cli).

    ```sh
    index.html
    README.md
    ```

- Create dockerfile

  ```Dockerfile
    FROM node:latest
    LABEL description="A demo Dockerfile for build Docsify."
    WORKDIR /docs
    RUN npm install -g docsify-cli@latest
    EXPOSE 3000/tcp
    ENTRYPOINT docsify serve .
  
  ```

  So, current directory structure should be this: 

  ```sh
   index.html
   README.md
   Dockerfile
  ```

- Build docker image

  ```sh
  docker build -f Dockerfile -t docsify/demo .
  ```

- Run docker image

  ```sh
  docker run -itp 3000:3000 --name=docsify -v $(pwd):/docs docsify/demo 
  ```

