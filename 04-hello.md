# Criando um Belo Hello World

Primeiramente vamos instalar o `expo-cli` globalmente e iniciar um projeto de exemplo, para isso, abra o prompt de comandos (cmd) e execute os comandos a seguir:

```bash
C:\Users\nome> npm install -g expo-cli
...
C:\Users\nome> expo --version
2.2.5
C:\Users\nome>mkdir projects
C:\Users\nome> cd projects
C:\Users\nome\projects> expo init HelloWorld
? Choose a template: blank
[19:30:41] Extracting project files...
[19:31:48] Customizing project...

Your project is ready at C:\Users\nome\projects\HelloWorld
To get started, you can type:

  cd HelloWorld
  expo start

C:\Users\nome\projects> cd HelloWorld
```

Opcionalmente, ao invés de utilizar o próprio celular, pode ser utilizado o emulador do Android.

Para isso, abra o AndroidStudio e clique em:

- Tools -> AVD Manager -> Actions | Play

## Ativando a depuração USB - Somente para celular Android

- Primeiro, abra "Configurações"
- Role a tela até encontrar "Sistema"
- Role a tela até encontrar "Sobre o dispositivo"
- Role para baixo novamente e encontre a entrada com o número da versão ou compilação (build number).
- Comece a tocar na seção "Número da versçai", e agora o Android exibirá uma mensagem informando que, em x cliques, você se tornará um desenvolvedor. Continue tocando até que o processo esteja concluído.
- Volte até a tela "Sistema"
- Role a tela até encontrar "Programador"
- Role a tela até encontrar "Depuração USB" e deixe a opção ativa

<!-- TODO: comandos adb device -->

## Ajustando o Hello World

Abra o VS Code e clique em: File -> Open Folder e abra a pasta do projeto criado.

Ative o terminal do VS Code: Terminal -> New Terminal

Agora, vamos executar a aplicação recém criada:

```bash
PS C:\Users\nome\projects\HelloWorld> expo start
[18:09:08] Starting project at C:\Users\nome\projects\HelloWorld
[18:09:09] Expo DevTools is running at http://localhost:19002
[18:09:09] Opening DevTools in the browser... (press shift-d to disable)
[18:09:20] Starting Metro Bundler on port 19001.
[18:09:26] Successfully ran `adb reverse`. Localhost URLs should work on the connected Android device.
[18:09:27] Tunnel ready.

  exp://169.254.186.144:19000

  To run the app with live reloading, choose one of:
  • Sign in as @fulano in Expo Client on Android or iOS. Your projects will automatically appear in the "Projects" tab.
  • Scan the QR code above with the Expo app (Android) or the Camera app (iOS).
  • Press a for Android emulator.
  • Press e to send a link to your phone with email/SMS.

Press ? to show a list of all available commands.
Logs for your project will appear below. Press Ctrl+C to exit.
```

Deve ser aberta uma página da Web com a interface do Expo, escolha a opção "Local", use a aplicação expo para escanear o QR Code que é exibido.

Agora, vamos customizar a tela da aplicação que está sendo exibida:

```jsx
// App.js
import * as React from "react";
import { Text, View, StyleSheet } from "react-native";
import { Constants } from "expo";

export default class App extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.paragraph}>Olá Mundo!</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    paddingTop: Constants.statusBarHeight,
    backgroundColor: "#ecf0f1",
    padding: 8
  },
  paragraph: {
    margin: 24,
    fontSize: 18,
    fontWeight: "bold",
    textAlign: "center"
  }
});
```

## Conhecendo o React Native

### Criando componentes

Vamos colocar uma segunda frase em nossa tela:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.paragraph}>Olá Mundo!</Text>
        <Text style={styles.paragraph}>
          Vamos começar a aprender React Native?
        </Text>
      </View>
    );
  }
}
// Código posterior omitido
```

E se quiséssemos utilizar o mesmo estilo em outras partes de nosso sistema? Para que isso fique mais fácil podemos criar um componente.

Vamos criar uma pasta chamada components e dentro dela um arquivo chamado Mensagem.js:

```jsx
// components/Mensagem.js
import * as React from "react";
import { StyleSheet, Text } from "react-native";

export class Mensagem extends React.Component {
  render() {
    return (
      <Text style={styles.paragraph}>
        Nosso primeiro componente React Native!
      </Text>
    );
  }
}

const styles = StyleSheet.create({
  paragraph: {
    margin: 24,
    fontSize: 18,
    fontWeight: "bold",
    textAlign: "center"
  }
});
```

Agora, no arquivo App.js podemos importar o componente que foi criado anteriormente:

```jsx
// App.js
// Código anterior omitido

// Novidade aqui!
import { Mensagem } from "./components/Mensagem";

export default class App extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.paragraph}>Olá Mundo!</Text>

        {/* Novidade aqui! */}
        <Mensagem />
      </View>
    );
  }
}
// Código posterior omitido
```

### Passando props

A maioria dos componentes pode ser personalizada quando eles são criados, com diferentes parâmetros. Esses parâmetros de criação são chamados de _props_.

Nosso componente está com o texto "Fixo", o ideal é que o texto da mensagem seja parametrizado, vamos fazer isso agora:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.paragraph}>Olá Mundo!</Text>

        {/* Novidade aqui! */}
        <Mensagem texto="Passando propriedades dinamicamente!" />
      </View>
    );
  }
}
// Código posterior omitido
```

```jsx
// components/Mensagem.js
// Código anterior omitido
export class Mensagem extends React.Component {
  render() {
    return (
      <Text style={styles.paragraph}>
        {/* Novidade aqui! */}
        {this.props.texto}
      </Text>
    );
  }
}
// Código posterior omitido
```

As _props_ permitem criar um único componente que é usado em muitos lugares diferentes do aplicativo, com propriedades ligeiramente diferentes em cada lugar. Basta se referir a `this.props` em sua função de renderização.

Será então que podemos tornar nosso código mais enxuto? Vamos utilizar mais nosso componente de mensagens:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        {/* Novidade aqui! */}
        <Mensagem texto="Olá Mundo via props!" />
        <Mensagem texto="Passando propriedades dinamicamente!" />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    paddingTop: Constants.statusBarHeight,
    backgroundColor: "#ecf0f1",
    padding: 8
  }
});
// Código posterior omitido
```

### Setando state

Existem dois tipos de dados que controlam um componente: _props_ e _state_. _props_ são definidas pelo componente pai e são fixas durante todo o tempo de vida de um componente. Para os dados que vão mudar, temos que usar o _state_.

Em geral, você deve inicializar o estado no construtor e, em seguida, chamar `setState` quando quiser alterá-lo.

Vamos fazer um simples contator de cliques em nossa aplicação:

```jsx
// App.js
// Código anterior omitido
// Novidade aqui!
import { Button, StyleSheet, View } from "react-native";

export default class App extends React.Component {
  // Novidade aqui!
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };
  }

  // Novidade aqui!
  handleClick() {
    this.setState({
      clicks: this.state.clicks + 1
    });
  }

  render() {
    return (
      <View style={styles.container}>
        {/* Novidade aqui! */}
        <Button
          title={`Clicou ${this.state.clicks} vezes`}
          onPress={this.handleClick.bind(this)}
        />
      </View>
    );
  }
}
// Código posterior omitido
```

Outra forma de fazer o bind:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };

    // Novidade aqui!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({
      clicks: this.state.clicks + 1
    });
  }

  render() {
    return (
      <View style={styles.container}>
        {/* Novidade aqui! */}
        <Button
          title={`Clicou ${this.state.clicks} vezes`}
          onPress={this.handleClick}
        />
      </View>
    );
  }
}
// Código posterior omitido
```

E mais uma forma de fazer o bind:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };
  }

  // Novidade aqui!
  handleClick = () => {
    this.setState({
      clicks: this.state.clicks + 1
    });
  };

  render() {
    return (
      <View style={styles.container}>
        <Button
          title={`Clicou ${this.state.clicks} vezes`}
          onPress={this.handleClick}
        />
      </View>
    );
  }
}
// Código posterior omitido
```

setState também possui uma segunda forma onde é passada uma função como argumento:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };
  }

  // Novidade aqui!
  handleClick = () => {
    this.setState(prevState => ({ clicks: prevState.clicks + 1 }));
  };

  render() {
    return (
      <View style={styles.container}>
        <Button
          title={`Clicou ${this.state.clicks} vezes`}
          onPress={this.handleClick}
        />
      </View>
    );
  }
}
// Código posterior omitido
```

### Ciclo de vida

<!-- https://code.likeagirl.io/understanding-react-component-life-cycle-49bf4b8674de -->

Cada componente React vem com vários métodos que permitem aos desenvolvedores atualizar o estado do aplicativo e refletir a alteração na interface do usuário. Existem três fases principais de um componente, incluindo mounting (montagem), updating (atualização) e unmounting (desmontagem).

#### Mounting

Esses métodos serão chamados quando uma instância de um componente React for criada e montada no DOM.

##### constuctor()

Esse método é chamado antes de um componente React ser montado. É essencial chamar `super(props)` antes de qualquer declaração no construtor. Isto ocorre pois nos permitirá chamar o construtor da classe pai e inicializar a si mesmo caso nossa classe estenda qualquer outra classe que tenha o próprio construtor.

O construtor é perfeito para inicializar o estado ou vincular os manipuladores de eventos à instância da classe. Por exemplo:

```jsx
constructor(props) {
  super(props);
  this.state = {
    count: 0,
    value: 'Hey There!',
  };
  this.handleClick = this.handleClick.bind(this);
};
```

O construtor não deve causar nenhum efeito colateral.

#### componentWillMount()

`componentWillMount` será chamado uma vez antes do componente ser montado e será executado antes da função de renderização.

#### componentDidMount()

Depois que um componente é montado, esse método é chamado. Este é o local certo para carregar qualquer dado do _endpoint_.

Chamar aqui `setState` irá disparar re-render, então use este método com cuidado.

### Updating

#### componentDidUpdate(prevProps, prevState, snapshot)

Este método será chamado após cada renderização ocorrer. Como esse método é chamado apenas uma vez após a atualização, é um local adequado para implementar quaisquer operações de efeitos colaterais.

### Unmounting

#### componentWillUnmount()

Quando um componente é desmontado ou destruído, este método será chamado. Este é um lugar para fazer alguma limpeza como:

- Invalidando temporizadores
- Cancelar qualquer pedido de rede
- Remover manipuladores de eventos
- Limpar todas as assinaturas

![React Life Cycle](reactlifecycle.jpeg)

### JSX

<!-- https://reactjs.org/docs/introducing-jsx.html -->

A sintaxe de tags curiosa que estamos utilizando não é JavaScript nem HTML.

É chamado de JSX e é uma extensão de sintaxe para JavaScript. Recomenda-se usá-lo com o React para descrever como deve ser a interface do usuário. O JSX pode lembrá-lo de uma linguagem de marcação, mas ela vem com todo o poder do JavaScript.

O JSX produz React "elements".

#### Por que o JSX?

O React adota o fato de que a lógica de renderização é inerentemente associada a outra lógica da interface do usuário: como os eventos são tratados, como o estado muda com o tempo e como os dados são preparados para exibição.

Em vez de separar artificialmente as tecnologias, colocando marcação e lógica em arquivos separados, o React separa as preocupações com unidades fracamente acopladas chamadas “componentes” que contêm ambos.

O React não exige o uso de JSX, mas a maioria das pessoas considera útil como um auxílio visual ao trabalhar com a interface do usuário dentro do código JavaScript.

#### Incorporando Expressões no JSX

No exemplo abaixo, nós declaramos uma variável chamada _nome_ e a usamos dentro do JSX, colocando-a entre chaves:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    const nome = "José";

    return (
      <View style={styles.container}>
        <Text>Olá {nome}</Text>
      </View>
    );
  }
}
// Código posterior omitido
```

Você pode colocar qualquer expressão JavaScript válida dentro das chaves no JSX. Por exemplo, 2 + 2, usuario.nome ou formatMessage(message) são todas expressões JavaScript válidas.

No exemplo abaixo, incorporamos o resultado de chamar uma função JavaScript, formatMessage(message).

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  formatMessage(message = "") {
    return message.toUpperCase();
  }

  render() {
    const nome = "José";

    return (
      <View style={styles.container}>
        <Text>Olá {this.formatMessage(nome)}</Text>
      </View>
    );
  }
}
// Código posterior omitido
```

#### JSX é uma expressão também

Após a compilação, as expressões JSX se tornam chamadas de função JavaScript regulares e são avaliadas como objetos JavaScript.

Isso significa que você pode usar o JSX dentro de instruções if e for loops, atribuí-lo a variáveis, aceitá-lo como argumentos e retorná-lo de funções:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  formatMessage(message = "") {
    let el;
    if (message.length > 2) {
      el = <Text>{message.toUpperCase()}</Text>;
    } else {
      el = <Text>{message}</Text>;
    }
    return el;
  }

  render() {
    const nome = "José";
    const saudacao = "oi";

    return (
      <View style={styles.container}>
        {this.formatMessage(saudacao)}
        {this.formatMessage(nome)}
      </View>
    );
  }
}
// Código posterior omitido
```

#### Especificando Atributos com JSX

Você pode usar aspas para especificar literais de string como atributos:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <View style={styles.container}>
        <Mensagem texto="Olá José" />
      </View>
    );
  }
}
// Código posterior omitido
```

Você também pode usar chaves para incorporar uma expressão JavaScript em um atributo:

```jsx
// App.js
// Código anterior omitido
export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    const minhaMsg = "Olá José";
    return (
      <View style={styles.container}>
        <Mensagem texto={minhaMsg} />
      </View>
    );
  }
}
// Código posterior omitido
```

### Componentes de Função e Classe

Já vimos como criar componentes a partir de uma classe (a classe Mensagem), porém, há uma forma simplificada de criar componentes através de funções:

```jsx
// App.js
// Código anterior omitido
function MensagemSimples(props) {
  return <Text>Uma componente de função: {props.texto}</Text>;
}

export default class App extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    const minhaMsg = "Olá José";
    return (
      <View style={styles.container}>
        <MensagemSimples texto={minhaMsg} />
      </View>
    );
  }
}
// Código posterior omitido
```

Essa função é um componente React válido porque aceita um único argumento de objeto "props" (que significa propriedades) com dados e retorna um elemento React. Chamamos esses componentes de "componentes de função" porque são literalmente funções JavaScript.

## Instalando Expo Client para desenvolvimento e debug da aplicação

O Expo Client permite que você teste o código diretamente em um dispositivo físico, para isso, ele deve ser instalado através da
[loja de aplicativos](https://play.google.com/store/apps/details?id=host.exp.exponent&referrer=www).

Uma grande vantagem do Expo Client é que podemos compilar o código em um computador Windows/Linux e executar em um iPhone.

## Analizando Expo Client & Dicas

Ao balançar o dispositivo, podemos acessar o menu do desenvolvedor do Expo Cli, onde podemos:

- Reload: Recarregar a aplicação
- Debug JS Remoately: Debugar a aplicação remotamente (Chrome remote debugging)
- Habilitar/Desabilitar o Live Reload
- Habilitar o Hot Reloading
- Ativar o "inspector"
- Exibir o monitor de performance
- Iniciar/Parar o profiler de JavaScript

## Buildando um APK ou IPA via Expo CLi

A construção do binário, em sua versão final que será entregue para a loja de aplicativos é realizada no servidor do Expo:

![](assets/expowf.png)

<!--
https://docs.expo.io/versions/latest/distribution/building-standalone-apps

https://docs.expo.io/versions/v24.0.0/expokit/detach
-->

Deve ser impostado o seguinte commando:

```bash
PS C:\Users\usuario\projects\HelloWorld> expo build:android
```
