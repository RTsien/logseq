blog:: true
status:: [[TODO]]
updated:: [[2025-10-25]]
created:: [[2025-10-16]]

- logseq github action -> richie.wiki
- {{embed ((68e8ca1d-b3ce-4019-8a57-22ddcb36f1ec))}}
-
- 改造property过滤规则: https://github.com/RTsien/logseq/tree/wiki-0.10.6
	- git clone 到跟graph同级目录
- 改造pubish-spa使用logseq-mod: https://github.com/RTsien/publish-spa/tree/wiki-0.3.1
	- 具体操作参考pubish-spa readme
-
-
-
- 直接粘贴gh-page分支链接给vercel就行了
-
- logseq graph仓库添加`.github/workflows/publish.yml`
  ```yml
  on: [push]
  
  permissions:
    contents: write
  jobs:
    test:
      runs-on: ubuntu-latest
      name: Publish Logseq graph
      steps:
        - uses: actions/checkout@v4
        - uses: rtsien/publish-spa@wiki-0.3.1
        - name: Add a nojekyll file # to make sure asset paths are correctly identified
          run: touch $GITHUB_WORKSPACE/www/.nojekyll
        - name: Deploy 🚀
          uses: JamesIves/github-pages-deploy-action@v4
          with:
            folder: www
  ```