Para construir usando o docker:

```bash
> docker run --rm -v "$PWD:/gitbook" -p 4000:4000 billryan/gitbook gitbook pdf .
```

Para construir localmente:

```bash
> npm install -g gitbook-cli
```

É necessário instalar o software Calibre: https://calibre-ebook.com/download

No Linux:
```bash
> apt-get install calibre
```

Para compilar e abrir a apostila no Windows:
```bash
> npm start; start "ApostilaRN.pdf"
```
