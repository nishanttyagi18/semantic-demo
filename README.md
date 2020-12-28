## 🚀semantic-release

> **Semantic Release** is an Open-Source Software tool for automatically versioning your software with Semantic Versions based on your Git commit messages.

_Features of semantic-release_

- Automatically versioning your software.
- Generates release notes and maintains a `CHANGELOG.md` file
- Upgrade the version based on commit messages.
- It expects the commits to be Conventional commit.

##### Setting up the project

---

- Install the following dependencies

  ```
  semantic-release
  @semantic-release/commit-analyzer
  @semantic-release/release-notes-generator
  @semantic-release/changelog
  @semantic-release/git
  @semantic-release/gitlab
  conventional-changelog-conventionalcommits
  ```

- change the name of your project in `package.json` as `@gitlab_username/project_name` or `@groupname/project_name`

- create a config file for semantic-release `release.config.js` and configure the plugins you want to use. A dummy file for this would be like this. You can change it according to your needs.

  ```
  module.exports = {
      "verifyConditions": [],
      plugins: [
          [
              "@semantic-release/commit-analyzer",
              {
                  preset: "conventionalcommits",
              },
          ],
          '@semantic-release/release-notes-generator',
          ['@semantic-release/changelog',
              {
                  changelogFile: "docs/CHANGELOG.md",
              }
          ],
          "@semantic-release/npm",
          [
              "@semantic-release/git",
              {
                  assets: ["docs", "dist/**/*.{js,css}"],
              },
          ],
          '@semantic-release/gitlab'
      ],
      branches: [
          'master',
          'dev'
      ],
      dryRun: false
  }
  ```

- Now we have to configure the CI file `.gitlab-ci.yml`. Here we will specify the `npx semantic-release` in the script so that whenever we push the code to gitlab, `semantic-release` will run and upgrade the version

- Here we will also specify the `CHANGELOG.md` file so that changes made to the project will be reflected to this file.

- The `.gitlab-ci.yml` file will look like this. Again, you can configure this also according to your project requirements.

  ```
  stages:
  - release

  release:
  image: node:13
  stage: release
  only:
      refs:
      - master
      - alpha

  script:
      - touch CHANGELOG.md
      - npm install @semantic-release/gitlab @semantic-release/changelog
      - npx semantic-release
  artifacts:
      paths:
      - CHANGELOG.md
  ```

---

Okay, the local part is done. Now head over to your project's repo on gitlab. we have to configure some things here.

- Go to your profile section.
- open settings.
- on the left sidebar you will see _Access tokens_
- Here you have to name your token _(it can be anything)_ and specify the _expiry date_, also give the access to _API_.
- Copy the token and save it somewhere we will need it.
- Now, go to your _project settings > CI/CD_
- In the variable section, add a new variable named with `GITLAB_TOKEN` and paste the token you copied from profile.
- you are all set!.

Make commit and push the code. Don't forget to use `conventional-commit` format. And done!. You can see the new version in the Release section with CHANGELOG.

#### Publishing to NPM

> If you want to publish the project on npm also then you have to do some extra steps. Here i will walk you through them.

After doing the above steps you need to follow the below part for publishing to npm.

- create a file `.npmrc` and add the below code to it :

  ```
  # Set URL for your scoped packages
  # For example, a package named `@foo/bar` uses this URL for download
  npm config set @GITLAB_USERNAME:registry http://gitlab.com/api/v4/packages/npm/

  # Add the token for the scoped packages URL
  # Use this to download `@foo/` packages from private projects
  npm config set '//gitlab.com/api/v4/packages/npm/:_authToken' "YOUR_GITLAB_TOKEN"

  # Add token for to publish to the package registry
  # Replace <your_project_id> with the project you want to publish your package to
  npm config set '//gitlab.com/api/v4/projects/PROJECT_ID/packages/npm/:_authToken' "YOUR_GITLAB_TOKEN"
  ```

  you have to change `GITLAB_USERNAME`, `YOUR_GITLAB_TOKEN`, `PROJECT_ID`according to yours.

* Come to `package.json`
  - Change the package name to `@NPM_USERNAME/PROJECT_NAME`
  - for publishing to npm you need to install one more package : `@semantic-release/npm`
  - add the follwing code to `package.json`:
    ```
    "publishConfig": {
            "@GITLAB_USERNAME:registry": "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/npm/"
        }
    ```
* Now, come to `release.config.js`
  - Add following plugin to the file:
    ```
    "@semantic-release/npm"
    ```
* That's it! commit the changes and run the following command :
  `npm publish --access public`
  Remember, first you need to login with the npm account locally then run the above command. This will publish the npm package _(This need to be done only once, just to tell npm that this package is public)_.
* To allow gitlab to access npm account and publish the package you need yo add `NPM_TOKEN` to the CI/CD variables. you can generate it from your npm profile.
* Now you are all set! whenever you will make any changes to the project and push it, it will publish a version to GITLAB and then to NPM.
