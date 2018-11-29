# Começando com o React Native

## Recaptulando sobre o React

O React é uma biblioteca JavaScript declarativa, eficiente e flexível para criar interfaces com o usuário. Ele permite compor interfaces de usuário complexas a partir de pequenos e isolados códigos chamados “componentes”.

```jsx
class ListaCompras extends React.Component {
  render() {
    return (
      <div className="lista-compras">
        <h1>Lista de Compras para: {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}
```

Utiliza-se componentes para dizer ao React o que será exibido na tela. Quando os dados forem alterados, o React atualizará e renderizará novamente com eficiência os componentes.

Aqui, o ListaCompras é uma classe de componente React ou o tipo de componente React. Um componente recebe parâmetros, chamados props (abreviação de "propriedades"), e retorna uma hierarquia de componentes visuais para exibir através do método "render".

O método render retorna uma descrição do que se deseja ver na tela. React pega a descrição e exibe o resultado. Em particular, render retorna um elemento React, que é uma descrição simples do que renderizar. A maioria dos desenvolvedores do React usa uma sintaxe especial chamada “JSX”, que facilita a gravação dessas estruturas. A sintaxe <div\/> é transformada no momento da criação para React.createElement ('div'). O exemplo acima é equivalente a:

```jsx
return React.createElement(
  "div",
  { className: "lista-compras" },
  React.createElement("h1" /* ... filhos de h1 ... */),
  React.createElement("ul" /* ... filhos de ul ... */)
);
```

O JSX possui todo o poder do JavaScript. Coloca-se qualquer expressão JavaScript dentro de chaves dentro do JSX. Cada elemento React é um objeto JavaScript que se pode armazenar em uma variável ou passar ao programa.

O componente ListaCompras acima apenas renderiza componentes DOM internos, como ```<div>``` e ```<li>```. Mas também pode compor e renderizar componentes React personalizados. Por exemplo, podemos nos referir a toda a lista de compras escrevendo ```<ListaCompras/>```. Cada componente React é encapsulado e pode operar de forma independente; Isso permite que se construa interfaces com o usuário complexas a partir de componentes simples.

Podemos testar a compilação de código JSX de forma online através do link: https://babeljs.io/repl/

## Recapitulando sobre o Redux

O Redux é um contêiner de estado previsível para aplicativos JavaScript.

Ele ajuda escrever aplicativos que se comportam de maneira consistente, executados em diferentes ambientes (cliente, servidor e nativo) e são fáceis de testar. Além disso, proporciona uma ótima experiência de desenvolvedor, como edição de código ao vivo combinada com um depurador de viagem no tempo.

O Redux pode ser utilizado junto com o React ou com qualquer outra biblioteca de visualizações. E é uma biblioteca pequena (2kB, incluindo dependências).

### Conceitos centrais

Imagine que o estado de um aplicativo é descrito como um objeto simples. Por exemplo, o estado de um aplicativo todo pode ter esta aparência:

```jsx
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

Este objeto é como um "modelo", exceto que não há "setters". Isso acontece para que diferentes partes do código não alterem o estado arbitrariamente, causando bugs difíceis de reproduzir.

Para mudar algo no estado, você precisa despachar uma ação. Uma ação é um objeto JavaScript simples (observe como não introduzimos nenhuma mágica?) Que descreva o que aconteceu. Aqui estão algumas ações de exemplo:

```jsx
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

Obrigando que todas as alterações são descritas como uma ação nos permite ter uma compreensão clara do que está acontecendo no aplicativo. Se algo mudou, sabemos porque mudou. Ações são como rastros do que aconteceu. Finalmente, para amarrar estados e ações juntos, escrevemos uma função chamada "reducer". Novamente, nada de mágico - é apenas uma função que toma o estado e a ação como argumentos e retorna o próximo estado do aplicativo. Seria difícil escrever uma função desse tipo para um aplicativo grande, por isso escrevemos funções menores gerenciando partes do estado:

```jsx
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter
  } else {
    return state
  }
}
​
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([{ text: action.text, completed: false }])
    case 'TOGGLE_TODO':
      return state.map(
        (todo, index) =>
          action.index === index
            ? { text: todo.text, completed: !todo.completed }
            : todo
      )
    default:
      return state
  }
}
```

E escrevemos outro redutor que gerencia o estado completo do nosso aplicativo chamando esses dois redutores para as chaves de estado correspondentes:

```jsx
function todoApp(state = {}, action) {
  return {
    todos: todos(state.todos, action),
    visibilityFilter: visibilityFilter(state.visibilityFilter, action)
  };
}
```

Esta é basicamente a ideia do Redux. Observe que não usamos nenhuma API do Redux. Ele vem com alguns utilitários para facilitar esse padrão, mas a idéia principal é que se descreva como o estado é atualizado ao longo do tempo em resposta a objetos de ação, e 90% do código que você escreve é ​​simplesmente JavaScript, sem uso do Redux em si, suas APIs ou qualquer "mágica".

### Os três princípios

#### Única fonte de verdade

"O estado de todo o aplicativo é armazenado em uma árvore de objetos em um único local de armazenamento."

Isso facilita a criação de aplicativos universais, já que o estado do servidor pode ser serializado e implantado no cliente sem nenhum esforço extra de codificação. Uma única árvore de estado também facilita a depuração ou inspeção de um aplicativo; Ele também permite que você persista o estado do seu aplicativo em desenvolvimento, para um ciclo de desenvolvimento mais rápido. Algumas funcionalidades que têm sido tradicionalmente difíceis de implementar - Desfazer / Refazer, por exemplo - podem subitamente tornar-se triviais de implementar, se todo o seu estado estiver armazenado em uma única árvore.

```jsx
console.log(store.getState())
​
/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

#### Estado é somente leitura

"A única maneira de mudar o estado é emitir uma ação, um objeto descrevendo o que aconteceu."

Isso garante que nem as exibições nem os retornos de chamada da rede jamais serão gravados diretamente no estado. Em vez disso, eles expressam a intenção de transformar o estado. Como todas as mudanças são centralizadas e acontecem uma a uma em uma ordem estrita, não há condições corrida a serem observadas. Como as ações são apenas objetos simples, elas podem ser registradas, serializadas, armazenadas e, posteriormente, reproduzidas para fins de depuração ou teste.

```jsx
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})
​
store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

#### As alterações são feitas com funções puras

"Para especificar como a árvore de estados é transformada por ações, você escreve redutores puros."

Redutores são apenas funções puras que tomam o estado anterior e uma ação, e retornam o próximo estado. Lembre-se de retornar novos objetos de estado, em vez de alterar o estado anterior. Você pode começar com um único redutor e, à medida que seu aplicativo cresce, divida-o em redutores menores que gerenciam partes específicas da árvore de estados. Como os redutores são apenas funções, você pode controlar a ordem na qual eles são chamados, passar dados adicionais ou até mesmo fazer reduções reutilizáveis ​​para tarefas comuns, como paginação.

```jsx
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}
​
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}
​
import { combineReducers, createStore } from 'redux'
const reducer = combineReducers({ visibilityFilter, todos })
const store = createStore(reducer)
```

## Comparando React com React Native

<!-- https://medium.com/@alexmngn/from-reactjs-to-react-native-what-are-the-main-differences-between-both-d6e8e88ebf24 -->

### Configuração e Construção

React-Native é um framework e o ReactJS é uma biblioteca de JavaScript. Quando se inicia um novo projeto com o ReactJS, deverá ser escolhido um bundler como o Webpack que tentará descobrir quais módulos de empacotamento são necessários para o seu projeto. O React-Native vem com tudo o que se precisa. Quando se inicia um novo projeto React Native é fácil de configurar: leva apenas uma linha de comando para ser executada no terminal e está pronto para começar. Pode-se começar a codificar o primeiro aplicativo nativo imediatamente usando o ES6, alguns recursos do ES7 e até mesmo alguns polyfills.

Para executar o aplicativo, precisará ter o Xcode (para iOS, somente no Mac) ou o Android Studio (para Android) instalado em seu computador. Pode-se optar por executá-lo em um simulador / emulador da plataforma que deseja usar ou diretamente em seus próprios dispositivos.

### DOM e Estilização

O React-Native não usa HTML para renderizar o aplicativo, mas fornece componentes alternativos que funcionam de maneira semelhante. Esses componentes React-Native mapeiam os componentes reais reais da interface iOS ou Android que são renderizados no aplicativo.

A maioria dos componentes fornecidos pode ser traduzida para algo semelhante em HTML, onde, por exemplo, um componente View é semelhante a uma tag div e um componente Text é semelhante a uma tag p.

```jsx
import React, { Component } from "react";
import { View, Text } from "react-native";

export default class App extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.intro}>Hello world!</Text>
      </View>
    );
  }
}
```

Para estilizar os componentes React-Native, é necessário criar folhas de estilo em JavaScript.

```jsx
const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  content: {
    backgroundColor: "#fff",
    padding: 30
  },
  button: {
    alignSelf: "center",
    marginTop: 20,
    width: 100
  }
});
```

## Introdução ao React Native

<!-- http://engineering.monsanto.com/2018/01/11/react-native-intro/ -->

O React Native é uma solução multi-plataforma para escrever aplicativos móveis nativos. O Facebook abriu o código fonte do React Native em março de 2015. Eles o construíram porque, como muitas empresas, o Facebook precisava disponibilizar seus produtos na Web, bem como em várias plataformas móveis, e é difícil manter equipes especializadas necessárias para construir um mesmo app em diferentes plataformas. Depois de experimentar várias técnicas diferentes, a React Native foi a solução do Facebook para o problema.

### O que faz o React Native Diferente?

Já existem soluções para criar aplicativos para dispositivos móveis: desde escrever código nativo em linguagens proprietárias até escrever “aplicativos da Web para dispositivos móveis” ou soluções híbridas. Então, por que os desenvolvedores precisam de outra solução? Por que eles deveriam dar uma chance ao React Native?

Ao contrário de outras opções disponíveis, o React Native permite que os desenvolvedores escrevam aplicativos nativos no iOS e no Android usando JavaScript com o React em uma única base de código. Ele usa os mesmos princípios de design usados ​​pelo React na Web e permite criar interfaces usando o modelo de componente que já é familiar aos desenvolvedores. Além disso, ao contrário de outras opções que permitem usar tecnologias da Web para criar aplicativos híbridos, o React Native é executado no dispositivo usando os mesmos blocos de construção fundamentais usados ​​pelas soluções específicas da plataforma, tornando-a uma experiência mais natural para os usuários.

## Introdução ao Expo Platform

<!-- https://expo.io/ -->

Expo é um conjunto de ferramentas gratuitas e de código aberto criadas em torno do React Native para ajudá-lo a criar projetos iOS e Android nativos usando JavaScript e React.

Os aplicativos Expo são aplicativos React Native que contêm o Expo SDK. O SDK é uma biblioteca nativa e JS que fornece acesso à funcionalidade do sistema do dispositivo (como câmera, contatos, armazenamento local e outros hardwares). Isso significa que você não precisa usar o Xcode ou o Android Studio ou escrever qualquer código nativo, além de tornar seu projeto puro JavaScript, ou seja, muito portável, pois pode ser executado em qualquer ambiente nativo que contenha o Expo SDK.

O Expo também fornece componentes de interface do usuário para lidar com uma variedade de casos de uso que quase todos os aplicativos cobrirão, mas não serão integrados ao núcleo React Native, por exemplo, ícones, desfocar visões e muito mais.

Finalmente, o Expo SDK fornece acesso a serviços que normalmente são difíceis de gerenciar, mas são exigidos por quase todos os aplicativos. O mais popular entre estes: o Expo pode gerenciar os assets, pode cuidar de notificações push, e pode construir binários nativos que estão prontos para implantar na loja de aplicativos.

# Laboratório: Criando o projeto React Native: Sandbox

Baixar a aplicação Expo para o seu celular (Android ou iOs)

Acessar o endereço https://snack.expo.io/ e inserir o código abaixo:

```jsx
import * as React from "react";
import { Text, View } from "react-native";

export default class App extends React.Component {
  render() {
    return (
      <View style={{ marginTop: 50 }}>
        <Text>Olá Mundo!</Text>
      </View>
    );
  }
}
```

Clicar em "Run", selecionar QR Code e escanear o QR Code com o App Expo do Celular.
