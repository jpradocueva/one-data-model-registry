# One Data Model Registry

[One Data Model Registry](https://one-data-model.github.io/prototype-registry/)


### Update JSON file
1. `data/convertjson.json`stores the content of the registry.
2. Each time that a new registration is introduced this fime MUST be updated.

### Run In Development - On Terminal
Before rendering the GitHub pages the dynamic table MUST be `build`
To do that follow this process:

`1. npm install`

`2. npm audit fix`
(if needed)

`3. npm run-script build`

`4. npm run-script build:server`

`5. npm run-script server`

Go to Localhost:9088 in browser to verify that the new content is available.

### GitHub pages
Now the dynamic pages are `build` and stored in:
* `/dev/server/dist/build.js`, and
* `/dev/server/dist/build.js.map`

1. `Push` these two files to the `gh-pages` of the official registry.
2. Refresh `gh-pages` by forcing it to be build again.
3. New content is lived in the GitHub Pages.

