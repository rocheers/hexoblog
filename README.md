# Yuan Zhang's personal website

- framework: Hexo
- theme: [material-flow](https://github.com/stkevintan/hexo-theme-material-flow)

## Installation

```bash
npm i -S hexo-generator-search hexo-generator-feed hexo-renderer-less hexo-autoprefixer hexo-generator-json-content
```

---
## Updates

### 02.10.2017
To support LaTex, I choose `hexo-math` plugin to help me represent math expression in blog.

```bash
npm i -S hexo-math
```

**P.S.:**  Technically, it can be used after `hexo-math` installed without any change in `_config.yml`. However, sometimes it doesn't work properly. If it happens, run the following script:
```bash
npm i -S hexo-renderer-mathjax
```