# Desenvolvendo uma versão mobile para Controle de Poneys

## Splash Screen

![Splash Screen](assets/splash300.png)

## Tela Principal

![Tela Principal](assets/mainscreen300.png)

## Criando o Projeto CoponeyMob

Agora que já conhecemos as estruturas básicas do React Native, vamos criar nosso projeto que será um CRUD (Create/Retrieve/Update/Delete) de poneys.

O primeiro passo é a criação do projeto em si:

```bash
PS C:\Users\nome\projects> expo init CoponeyMob
? Choose a template: blank
[10:20:22] Extracting project files...
[10:22:13] Customizing project...

Your project is ready at C:\Users\nome\projects\CoponeyMob
To get started, you can type:

  cd CoponeyMob
  expo start

PS C:\Users\nome\projects> cd .\CoponeyMob\
PS C:\Users\nome\projects\CoponeyMob> expo start
[10:43:15] Starting project at C:\Users\nome\projects\CoponeyMob
[10:43:16] Expo DevTools is running at http://localhost:19002
[10:43:16] Opening DevTools in the browser... (press shift-d to disable)
...
```

## Colocando os binários do Android no PATH

Caso os comandos `adb` e `emulator` não estejam disponíveis, verifique a configuração das variáveis de ambiente do Windows:

- Tecla Windows -> Editar as variáveis de ambiente do sistema -> Clique em Path e 'Editar'
- Certifique-se que os caminhos abaixo estão configurados:
  - %USERPROFILE%\AppData\Local\Android\Sdk\platform-tools
  - %USERPROFILE%\AppData\Local\Android\Sdk\emulator

## Executando o emulador a partir da linha de comandos:

Pode-se iniciar o emulador direto pela linha de comando para evitar a necessidade de abrir o Android Studio:

```bash
> emulator -list-avds
Nexus_5X_API_25

> emulator -avd Nexus_5X_API_25
```

O comando 'adb devices' lista os dispositivos conectados, tando emulador quanto físicos:

```bash
> adb devices
```

Em caso de dificuldade de conexão com o Expo local, pode ser feita tentativa de novo 'adb reverse':

```bash
> adb -s 0040483203 reverse tcp:19001 tcp:19001
```

É possível também ver o log do dispositivo pela linha de comandos

```bash
> adb logcat
```

## Adicionando Libs ao projeto

### Editorconfig

Criar na raiz do projeto o arquivo .editorconfig com o seguinte conteúdo:

```
[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8
```

### ESlint

Instalar as seguintes dependências:

npm install --save-dev babel-eslint prettier-eslint eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react

Criar na raiz do projeto o arquivo .eslintrc com o seguinte conteúdo:

```json
{
  "parser": "babel-eslint",
  "env": {
    "es6": true,
    "browser": true,
    "node": true
  },
  "plugins": ["react", "import", "jsx-a11y"],
  "extends": ["eslint:recommended", "plugin:react/recommended"],
  "globals": {
    "Expo": true
  }
}
```

Agora vamos fazer com que haja auto-formatação do código sempre que os arquivos do projeto forem salvos:

No VSCode acessar: File -> Preferences -> Settings -> Workspace Settings -> ... -> Open Settings.json e deixar o arquivo conforme abaixo:

```json
{
  // Format a file on save. A formatter must be available, the file must not be auto-saved, and editor must not be shutting down.
  "editor.formatOnSave": true,
  // Enable/disable default JavaScript formatter (For Prettier)
  "javascript.format.enable": false,
  // Use 'prettier-eslint' instead of 'prettier'. Other settings will only be fallbacks in case they could not be inferred from eslint rules.
  "prettier.eslintIntegration": true
}
```

Em seguida, reiniciar o VS Code.

### Reactotron

Primeiro, vamos instalar o software no desktop: https://github.com/infinitered/reactotron/releases.

Agora, precisamos incluir a dependência em nosso projeto e fazer o adb reverse da porta 9090 que será utilizada para comunicação:

```bash
PS C:\Users\nome\projects\CoponeyMob> npm install --save-dev reactotron-react-native
PS C:\Users\nome\projects\CoponeyMob> adb reverse tcp:9090 tcp:9090
```

Criar o arquivo de configuração do Reactotron na raiz do projeto:

```jsx
// ReactotronConfig.js
import Reactotron from "reactotron-react-native";

Reactotron.configure({ port: 9090 }) // controls connection & communication settings
  .useReactNative() // add all built-in react native plugins
  .connect(); // let's connect!
```

Ajustar o arquivo App.js:

```jsx
// App.js
import React from "react";
import { StyleSheet, Text, View } from "react-native";

// Novidade aqui
import "./ReactotronConfig";
import Reactotron from "reactotron-react-native";

export default class App extends React.Component {
  render() {
    // Novidade aqui
    Reactotron.log("Testando a conexão com o Reactotron.");

    return (
      <View style={styles.container}>
        <Text>Open up App.js to start working on your app!</Text>
      </View>
    );
  }
}
```

#### Para saber mais:

- https://github.com/infinitered/reactotron/blob/master/docs/quick-start-react-native.md

Sobre a questão de configuração da porta do Reactotron:

- https://github.com/infinitered/reactotron/issues/690

### NativeBase

A biblioteca NativeBase apresenta uma série de componentes visuais multiplataforma que nos auxilia no desenvolvimento de nossa aplicação.

Precisamos incluir a dependência em nosso projeto:

```bash
PS C:\Users\nome\projects\CoponeyMob> npm install --save native-base @expo/vector-icons
```

### Demais bibliotecas

As demais bibliotecas que utilizaremos em nosso projeto, serão instaladas durante o desenvolvimento das funcionalidades.

## Analizando package.json

Vamos atualizar o nome do projeto no arquivo package.json:

```json
{
  "name": "coponeymob",
  "main": "node_modules/expo/AppEntry.js",
  "private": true,
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "eject": "expo eject"
  },
  "dependencies": {
    "@expo/vector-icons": "^8.0.0",
    "expo": "^31.0.2",
    "native-base": "^2.8.1",
    "react": "16.5.0",
    "react-native": "https://github.com/expo/react-native/archive/sdk-31.0.0.tar.gz"
  },
  "devDependencies": {
    "babel-eslint": "^10.0.1",
    "babel-preset-expo": "^5.0.0",
    "eslint-plugin-import": "^2.14.0",
    "eslint-plugin-jsx-a11y": "^6.1.2",
    "eslint-plugin-react": "^7.11.1",
    "prettier-eslint": "^8.8.2",
    "reactotron-react-native": "^2.1.0"
  }
}
```

## Configurando a Estrutura do Projeto

Os arquivos relativos ao código fonte de nosso projeto ficarão em uma pasta 'src', e, dentro dela, teremos as pastas 'api', 'assets', 'components' e 'reducers'.

O React Native não nos obriga a seguir uma estrutura pré-definida de pastas no projeto, a excessão é o arquivo `App.js` que é o ponto de início de nossa aplicação.

Vamos baixar e descompactar dentro da pasta src os assets da aplicação: http://bit.ly/cpmassets

<!--
Link: https://www.dropbox.com/s/zffz1e1uq7ha8wf/coponeymob-assets.zip?dl=1
-->

Agora, vamos apagar a pasta assets da raiz do projeto e atualizar o arquivo app.json com o novo caminho da splash screen e do ícone da aplicação:

app.json

```json
{
  "expo": {
    "name": "CoponeyMob",
    "description": "This project is really great.",
    "slug": "CoponeyMob",
    "privacy": "public",
    "sdkVersion": "31.0.0",
    "platforms": ["ios", "android"],
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./src/assets/icon.png",
    "splash": {
      "image": "./src/assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "updates": {
      "fallbackToCacheTimeout": 0
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true
    }
  }
}
```

### Para saber mais

How To Structure a React Native App For Scale - https://medium.com/the-andela-way/how-to-structure-a-react-native-app-for-scale-a29194cd33fc
