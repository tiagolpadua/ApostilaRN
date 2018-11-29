# Criando a tela Home

## Listando os Poneis

Nossa primeira tarefa mover nossa base de código para dentro da pasta `src`, para isso, iremos alterar o arquivo `App.js` para apontar para um componente que criaremos dentro da pasta `src`:

```jsx
// App.js
import ListarPoneysScreen from "./src/components/ListarPoneysScreen";
export default ListarPoneysScreen;
```

Agora, dentro da pasta `src\components`, vamo criar a tela inicial de nosso sistema, que será uma listagem de poneis:

```jsx
// src/components/ListarPoneysScreen.js
import { Left, ListItem, Text } from "native-base";
import React from "react";
import { FlatList, StyleSheet, View } from "react-native";

class ListarPoneysScreen extends React.Component {
  poneys = [
    { nome: "Tremor" },
    { nome: "Tzar" },
    { nome: "Pégaso" },
    { nome: "Epona" },
    { nome: "Macedonio" },
    { nome: "Vicário" },
    { nome: "Tro" },
    { nome: "Nicanor" },
    { nome: "Niceto" },
    { nome: "Odón" },
    { nome: "Relâmpago" },
    { nome: "Pio" },
    { nome: "Elegante" },
    { nome: "Pompeu" }
  ];

  constructor(props) {
    super(props);
    this.poneys = this.poneys.map((p, idx) => ({ ...p, _id: idx + "" }));
  }

  render() {
    return (
      <View>
        <FlatList
          data={this.poneys}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text style={styles.item}>{item.nome}</Text>
              </Left>
            </ListItem>
          )}
          keyExtractor={item => item._id}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  itemContainer: {
    flex: 1,
    flexDirection: "row",
    borderBottomWidth: 1
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
    width: 120
  }
});

export default ListarPoneysScreen;
```

### Observação

Em relação ao warning:

```
Require cycles are allowed, but can result in uninitialized values. Consider refactoring to remove the need for a cycle.
- node_modules\expo\build\environment\logging.js:25:23 in warn
- node_modules\metro\src\lib\polyfills\require.js:109:19 in metroRequire
- node_modules\native-base\dist\src\basic\DatePicker.js:1:774 in <unknown>
...
```

Ele pode ser ignorado com segurança e já existe uma issue aberta na comunidade para corrigir: https://github.com/GeekyAnts/NativeBase/issues/2320

## Iniciando o Native Base

A biblioteca do native base necessita que sejam carregados alguns arquivos de fonte e ícones antes da exibição da tela inicial de nossa aplicação (http://docs.nativebase.io/docs/GetStarted.html), vamos criar um componente chamado `CoponeyMob` na pasta `src` que terá esta funcionalidade, vamos também ativar o Reactotron para o Coponeymob:

```jsx
// src/ReactotronConfig.js
import Reactotron from "reactotron-react-native";

Reactotron.configure({ port: 9090 }) // controls connection & communication settings
  .useReactNative() // add all built-in react native plugins
  .connect(); // let's connect!
```

```jsx
// src/CoponeyMob.js
import React from "react";
import { StyleSheet, View } from "react-native";
import Reactotron from "reactotron-react-native";
import ListarPoneysScreen from "./components/ListarPoneysScreen";
import "./ReactotronConfig";
import { Spinner } from "native-base";

Reactotron.log("Testando a conexão com o Reactotron.");

export default class CoponeyMob extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isReady: false
    };
  }

  async componentDidMount() {
    await Expo.Font.loadAsync({
      Roboto: require("native-base/Fonts/Roboto.ttf"),
      Roboto_medium: require("native-base/Fonts/Roboto_medium.ttf"),
      Ionicons: require("native-base/Fonts/Ionicons.ttf")
    });
    setTimeout(() => this.setState({ isReady: true }), 0);
  }

  render() {
    if (!this.state.isReady) {
      return (
        <View style={styles.loadingContainer}>
          <Spinner />
        </View>
      );
    } else {
      return <ListarPoneysScreen />;
    }
  }
}

const styles = StyleSheet.create({
  loadingContainer: {
    flexDirection: "column",
    justifyContent: "center",
    alignItems: "center",
    height: "100%"
  }
});
```

Devemos ajustar novamente nosso `App.js` para nossa nova tela inicial:

```jsx
// App.js
import CoponeyMob from "./src/CoponeyMob.js";
export default CoponeyMob;
```

## Criando uma rota para a tela

Primeiramente vamos instalar a biblioteca que fornece o recurso de navegação do react:

```bash
> npm install --save react-navigation
```

Nossa aplicação possuirá várias telas, temos que criar o mecanismo que permitirá a navegação entre estas telas, para isso, primeiro criaremos um novo componente que irá centralizar nosso fluxo de navegação.

```jsx
// src/CoponeyMobNav.js
import React from "react";
import { StyleSheet, View } from "react-native";
import { createStackNavigator } from "react-navigation";
import ListarPoneysScreen from "./components/ListarPoneysScreen";

const RootStack = createStackNavigator(
  {
    ListarPoneys: {
      screen: ListarPoneysScreen,
      navigationOptions: {
        title: "Lista de Poneys"
      }
    }
  },
  {
    initialRouteName: "ListarPoneys"
  }
);

class CoponeyMobNav extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <View style={styles.container}>
        <RootStack />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1
  }
});

export default CoponeyMobNav;
```

Em seguida, vamos alterar nosso componente principal para que exiba o componente de navegação:

```jsx
// src/CoponeyMob.js
// Código anterior omitido
  render() {
    if (!this.state.isReady) {
      return (
        <View style={styles.loadingContainer}>
          <Spinner />
        </View>
      );
    } else {
      // Novidade aqui
      return <CoponeyMobNav />;
    }
  }
// Código posterior omitido
```

A tela principal da aplicação deve estar com esta aparência:

![Lista de Poneys](assets/lista-nav300.png)

## Passando o controle de estado para o Redux

Para iniciarmos com a utilização do Redux, primeiro, precisamos instalar as bibliotecas necessárias:

```bash
> npm install --save redux react-redux
```

Agora vamos criar nosso primeiro reducer:

```jsx
// src/reducers/poneys.js
let poneys = [
  { nome: "Tremor" },
  { nome: "Tzar" },
  { nome: "Pégaso" },
  { nome: "Epona" },
  { nome: "Macedonio" },
  { nome: "Vicário" },
  { nome: "Tro" },
  { nome: "Nicanor" },
  { nome: "Niceto" },
  { nome: "Odón" },
  { nome: "Relâmpago" },
  { nome: "Pio" },
  { nome: "Elegante" },
  { nome: "Pompeu" }
];

poneys = poneys.map((p, idx) => ({ ...p, _id: idx + "" }));

const initialState = { list: poneys, viewDeleted: false };

export default function poneysReducer(state = initialState, action) {
  switch (action.type) {
    default:
      return state;
  }
}
```

Agora criaremos um arquivo `index.js` na pasta reducers que terá a função de combinar os vários reducers que criaremos:

```jsx
// src/reducers/index.js
import { combineReducers } from "redux";
import poneysReducer from "./poneys";

const rootReducer = combineReducers({
  poneys: poneysReducer
});

export default rootReducer;
```

Em seguira, criaremos um arquivo js que tem a função criar uma 'store' a partir dos 'reducuers':

```jsx
// src/configureStore.js.js
import { createStore } from "redux";
import rootReducer from "./reducers";

export default function configureStore() {
  let store = createStore(rootReducer);
  return store;
}
```

O próximo passo é configurar nosso componente principal para disponibilizar a 'store' em nossa árvore de componentes:

```jsx
// src/CoponeyMob.js
import { Spinner } from "native-base";
import React from "react";
import { StyleSheet, View } from "react-native";

// Novidades aqui
import { Provider } from "react-redux";
import configureStore from "./configureStore";

import Reactotron from "reactotron-react-native";
import CoponeyMobNav from "./CoponeyMobNav";
import "./ReactotronConfig";

Reactotron.log("Testando a conexão com o Reactotron.");

// Novidades aqui
const store = configureStore();

export default class CoponeyMob extends React.Component {
  // Código atual omitido

  render() {
    if (!this.state.isReady) {
      return (
        <View style={styles.loadingContainer}>
          <Spinner />
        </View>
      );
    } else {
      return (
        // Novidades aqui
        <Provider store={store}>
          <CoponeyMobNav />
        </Provider>
      );
    }
  }
}

// Código posterior omitido
```

Nossa tela de listagem de poneys deverá então obter a lista de poneys a partir do redux:

```jsx
// src/components/ListarPoneysScreen.js
import { Left, ListItem, Text } from "native-base";

// Novidade aqui
import PropTypes from "prop-types";

import React from "react";
import { FlatList, StyleSheet, View } from "react-native";

// Novidade aqui
import { connect } from "react-redux";

class ListarPoneysScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <View>
        <FlatList
         {/* Novidade aqui */}
          data={this.props.poneys.list}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text style={styles.item}>{item.nome}</Text>
              </Left>
            </ListItem>
          )}
          keyExtractor={item => item._id}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  itemContainer: {
    flex: 1,
    flexDirection: "row",
    borderBottomWidth: 1
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
    width: 120
  }
});

// Várias novidades aqui
const mapStateToProps = state => {
  return {
    poneys: state.poneys
  };
};

const mapDispatchToProps = {};

ListarPoneysScreen.propTypes = {
  poneys: PropTypes.object
};

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(ListarPoneysScreen);
```

### PropTypes

Conforme seu aplicativo cresce, você pode começar a enfrentar muitos bugs devido a problemas com tipos. Você pode usar extensões do JavaScript como Flow ou TypeScript para facilitar o trabalho com tipos em um projeto, porém, mesmo que você não os use, o React possui algumas ferramentas que permitem a checagem de tipos.

PropTypes possui uma gama de validadores que podem ser usados para garantir que os dados recebidos sejam válidos. Quando um valor inválido é fornecido para um prop, um aviso será mostrado no console do JavaScript. Por motivos de desempenho, propTypes é verificado apenas no modo de desenvolvimento.

## Botão para se Logar no sistema

Nosso botão de login/logout ficará no canto superior direito da aplicação, como haverão mais botões neste local, vamos criar um componente para contê-los e iniciar nossa lógica de login:

```jsx
// src/components/HeaderButtonsComponent.js
import {
  Button,
  Form,
  Icon,
  Input,
  Item,
  Label,
  Text,
  View
} from "native-base";
import React from "react";
import { Alert, Image, Modal, StyleSheet } from "react-native";

class HeaderButtonsComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      modalVisible: false,
      inputUserName: "",
      inputPassword: "",
      user: null
    };
  }

  closeLoginModal = () => {
    this.setState({ modalVisible: false });
  };

  openLoginModal = () => {
    this.setState({ modalVisible: true });
  };

  login = user => {
    this.setState({
      user
    });
  };

  logout = () => {
    this.setState({
      user: null
    });
  };

  handleLogin = () => {
    this.closeLoginModal();
    if (
      this.state.inputUserName === "admin" &&
      this.state.inputPassword === "123456"
    ) {
      this.login({ name: this.state.inputUserName });
    } else {
      Alert.alert("Erro", "Credenciais inválidas", [{ text: "OK" }]);
    }
  };

  handleLogout = () => {
    Alert.alert(
      "Logout",
      this.state.user.name + ", confirma o logout?",
      [{ text: "Sim", onPress: this.logout }, { text: "Não", style: "cancel" }],
      { cancelable: false }
    );
  };

  renderLoginModal = () => {
    return (
      <Modal
        animationType="slide"
        visible={this.state.modalVisible}
        onRequestClose={this.closeLoginModal}
      >
        <View>
          <Form>
            <Item floatingLabel>
              <Label>Usuário</Label>
              <Input
                autoCapitalize="none"
                onChangeText={inputUserName => this.setState({ inputUserName })}
                value={this.state.inputUserName}
              />
            </Item>
            <Item floatingLabel last style={{ marginBottom: 20 }}>
              <Label>Senha</Label>
              <Input
                secureTextEntry={true}
                autoCapitalize="none"
                onChangeText={inputPassword => this.setState({ inputPassword })}
                value={this.state.inputPassword}
              />
            </Item>
            <Button
              full
              primary
              style={{ marginBottom: 20 }}
              onPress={this.handleLogin}
            >
              <Text>Login</Text>
            </Button>
            <Button full light onPress={this.closeLoginModal}>
              <Text>Cancelar</Text>
            </Button>
          </Form>
        </View>
      </Modal>
    );
  };

  render() {
    return (
      <View style={styles.headerButtonContainer}>
        {this.state.user ? (
          <View style={styles.headerButtonContainer}>
            <Button transparent onPress={this.handleLogout}>
              <Image
                style={styles.headerIconMargin}
                source={require("../assets/admin.png")}
              />
            </Button>
          </View>
        ) : (
          <Button transparent onPress={this.openLoginModal}>
            <Icon
              style={[styles.headerIconFont, styles.headerIconMargin]}
              name="contact"
            />
          </Button>
        )}
        {this.renderLoginModal()}
      </View>
    );
  }
}

const styles = StyleSheet.create({
  headerButtonContainer: {
    flex: 1,
    flexDirection: "row"
  },
  headerIconMargin: {
    marginLeft: 10,
    marginRight: 10
  },
  headerIconFont: {
    fontSize: 35
  }
});

export default HeaderButtonsComponent;
```

O próximo passo é alterar o componente CoponeyMobNav para que ele exiba o botão HeaderButtonsComponent no cabeçalho da tela de listagem de poneis:

```jsx
// src/CoponeyMobNav.js
import React from "react";
import { StyleSheet, View } from "react-native";
import { createStackNavigator } from "react-navigation";
import ListarPoneysScreen from "./components/ListarPoneysScreen";

// Novidade aqui
import HeaderButtonsComponent from "./components/HeaderButtonsComponent";

const RootStack = createStackNavigator(
  {
    ListarPoneys: {
      screen: ListarPoneysScreen,
      navigationOptions: {
        title: "Lista de Poneys",

        // Novidade aqui
        headerRight: <HeaderButtonsComponent />
      }
    }
  },
  {
    initialRouteName: "ListarPoneys"
  }
);
// Código posterior omitido
```

## Mantento o estado de autenticação no Redux

Nossa próxima missão será exibir os botões de inclusão, alteração e exclusão de poneis, mas estes botões só podem ser exibidos se o usuário estiver logado, para facilitar que a informação do usuário autenticado esteja disponível em toda a nossa aplicação, devemos manter estas informações de autenciação em um estado gerido pelo redux.

Vamos criar um reducer que será responsável por gerenciar o profile do usuário logado:

```jsx
// src/reducers/profile.js
import { LOGIN, LOGOUT } from "../constants";

const initialState = {};

export default function profileReducer(state = initialState, action) {
  switch (action.type) {
    case LOGIN:
      return {
        user: action.data
      };
    case LOGOUT:
      return {};
    default:
      return state;
  }
}
```

Mas também teremos que criar um arquivo com as constantes que representam as ações:

```jsx
// src/constants.js
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";

export { LOGIN, LOGOUT };
```

Não podemos nos esquecer de alterar o arquivo de index.js que cria o `rootReducer` incluindo o `profileReducer` que criamos:

```jsx
// src/reducers/index.js
import { combineReducers } from "redux";
import poneysReducer from "./poneys";
import profileReducer from "./profile";

const rootReducer = combineReducers({
  profile: profileReducer,
  poneys: poneysReducer
});

export default rootReducer;
```

Em seguida, criaremos um arquivo contendo 'actions' que retornam objetos que serão utilizados pelo Redux e representam as ações que são realizadas sobre os estados:

```jsx
// src/actions.js
import { LOGIN, LOGOUT } from "./constants";

export function login(data) {
  return {
    type: LOGIN,
    data
  };
}

export function logout() {
  return {
    type: LOGOUT
  };
}
```

Pronto, agora já está tudo preparado para que o componente 'HederButtonsComponent' se ligue aos estados e ações do Redux:

```jsx
// src/components/HeaderButtonsComponent.js
// Código anterior omitido
import {
  Button,
  Form,
  Icon,
  Input,
  Item,
  Label,
  Text,
  View
} from "native-base";
import React from "react";
import { Alert, Image, Modal, StyleSheet } from "react-native";

// Novidades aqui
import PropTypes from "prop-types";
import { connect } from "react-redux";
import { bindActionCreators } from "redux";
import { login, logout } from "../actions";

class HeaderButtonsComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      modalVisible: false,
      inputUserName: "admin",
      inputPassword: "123456"
    };
  }

  closeLoginModal = () => {
    this.setState({ modalVisible: false });
  };

  openLoginModal = () => {
    this.setState({ modalVisible: true });
  };

  handleLogin = () => {
    this.closeLoginModal();
    if (
      this.state.inputUserName === "admin" &&
      this.state.inputPassword === "123456"
    ) {
      // Novidade aqui
      this.props.login({ name: this.state.inputUserName });
    } else {
      Alert.alert("Erro", "Credenciais inválidas", [{ text: "OK" }]);
    }
  };

  handleLogout = () => {
    Alert.alert(
      "Logout",

      // Novidade aqui
      this.props.profile.user.name + ", confirma o logout?",
      [
        // Novidade aqui
        { text: "Sim", onPress: this.props.logout },

        { text: "Não", style: "cancel" }
      ],
      { cancelable: false }
    );
  };

  // Código atual omitido

  render() {
    return (
      <View style={styles.headerButtonContainer}>
        {/* Novidades aqui */}
        {this.props.profile.user ? (
          <View style={styles.headerButtonContainer}>
            <Button transparent onPress={this.handleLogout}>
              <Image
                style={styles.headerIconMargin}
                source={require("../assets/admin.png")}
              />
            </Button>
          </View>
        ) : (
          <Button transparent onPress={this.openLoginModal}>
            <Icon
              style={[styles.headerIconFont, styles.headerIconMargin]}
              name="contact"
            />
          </Button>
        )}
        {this.renderLoginModal()}
      </View>
    );
  }
}

const styles = StyleSheet.create({
  headerButtonContainer: {
    flex: 1,
    flexDirection: "row"
  },
  headerIconMargin: {
    marginLeft: 10,
    marginRight: 10
  },
  headerIconFont: {
    fontSize: 35
  }
});

// Novidades aqui
HeaderButtonsComponent.propTypes = {
  profile: PropTypes.object,
  poneys: PropTypes.object,
  login: PropTypes.func,
  logout: PropTypes.func
};

const mapStateToProps = state => {
  return {
    profile: state.profile,
    poneys: state.poneys
  };
};

const mapDispatchToProps = dispatch =>
  bindActionCreators({ login, logout }, dispatch);

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(HeaderButtonsComponent);
```

## Botão pra Incluir Novo Poney

Agora que já temos o Redux controlando o estado da autenticação, nosso próximo passo é exibir um botão de inclusão de ponei caso o usuário esteja logado:

```jsx
// src/components/HeaderButtonsComponent.js
// Código anterior omitido
  render() {
    return (
      <View style={styles.headerButtonContainer}>
        {this.props.profile.user ? (
          <View style={styles.headerButtonContainer}>
            {/* Novidade aqui */}
            <Button transparent>
              <Icon
                style={[styles.headerIconFont, styles.headerIconMargin]}
                name="add"
                onPress={() =>
                  Alert.alert("Incluir", "Aqui irá a tela de incluir ponei", [
                    { text: "OK" }
                  ])
                }
              />
            </Button>
            <Button transparent onPress={this.handleLogout}>
              <Image
                style={styles.headerIconMargin}
                source={require("../assets/admin.png")}
              />
            </Button>
          </View>
        ) : (
          <Button transparent onPress={this.openLoginModal}>
            <Icon
              style={[styles.headerIconFont, styles.headerIconMargin]}
              name="contact"
            />
          </Button>
        )}
        {this.renderLoginModal()}
      </View>
    );
  }
// Código posterior omitido
```

## Botão pra Editar e Excluir Ponei

Temos mais dois botões a serem colocados na tela, mas estes vão ficar ao lado de cada ponei na listagem, são os botões de editar e excluir:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
  render() {
    return (
      <View>
        <FlatList
          data={this.props.poneys.list}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text style={styles.item}>{item.nome}</Text>
              </Left>
              <Right>
                {this.props.profile.user && (
                  <View style={{ flexDirection: "row", flex: 1 }}>
                    <Button
                      primary
                      onPress={() =>
                        Alert.alert(
                          "Alterar",
                          "Aqui irá a tela de Alterar ponei",
                          [{ text: "OK" }]
                        )
                      }
                      style={{ marginRight: 10 }}
                    >
                      <Icon name="create" />
                    </Button>
                    <Button
                      danger
                      onPress={() =>
                        Alert.alert(
                          "Excluir",
                          "Aqui exibirá a confirmação de exclusão do ponei",
                          [{ text: "OK" }]
                        )
                      }
                    >
                      <Icon name="trash" />
                    </Button>
                  </View>
                )}
              </Right>
            </ListItem>
          )}
          keyExtractor={item => item._id}
        />
      </View>
    );
  }
}

// Código atual omitido

const mapStateToProps = state => {
  return {
    poneys: state.poneys,

    // Novidade aqui
    profile: state.profile
  };
};

const mapDispatchToProps = {};

ListarPoneysScreen.propTypes = {
  poneys: PropTypes.object,

  // Novidade aqui
  profile: PropTypes.object
};

// Código posterior omitido
```

Mas apesar de nossa lógica estar aparentemente correta, os botões não são exibidos após o usuário se logar... Para que isso funcione, precisamos "avisar" a lista que ela precisa se atualizar:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
  render() {
    return (
      <View>
        <FlatList
          data={this.props.poneys.list}
          extraData={this.props.profile}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text style={styles.item}>{item.nome}</Text>
              </Left>
// Código posterior omitido
```

## Mapeamento do Serviço Rest para Listagem dos Poneys

O serviço REST que servirá de backend para nossa aplicação já se encontra pronto, sua URL base é https://coponeyapi.herokuapp.com. Os seguintes end-points estão disponíveis:

- GET/POST: https://coponeyapi.herokuapp.com/v1/poneys
- GET/PUT/DELETE: https://coponeyapi.herokuapp.com/v1/poneys/{id}

Caso queira dar uma olhada no código fonte deste serviço, ele pode ser acessado em https://github.com/tiagolpadua/CoponeyAPI.

O código de acionamento de nossa API ficará isolado em uma pasta 'api' dentro de 'src', mas para realizar os requests usaremos a biblioteca 'superagent', vamos instalá-la em nosso projeto:

```bash
> npm install --save superagent
```

Agora criaremos o arquivo que faz as chamadas para as APIs:

```jsx
// src/api/index.js
import request from "superagent";

// const URI = "http://localhost:3000/v1/poneys";
const URI = "https://coponeyapi.herokuapp.com/v1/poneys";

export function loadPoneysAPI() {
  return request.get(URI).set("Accept", "application/json");
}
```

O próximo passo é chamar esta API a partir do arquivo que define as ações:

```jsx
// src/actions.js
import { loadPoneysAPI } from "./api";
import { LOAD_PONEYS, LOGIN, LOGOUT } from "./constants";

export function loadPoneys() {
  loadPoneysAPI()
    .then(res => ({
      type: LOAD_PONEYS,
      data: res.body
    }))
    .catch(error => {
      alert(error.message);
    });
}

// Código posterior omitido
```

Devemos também ajustar o arquivo de constantes para incluir a nova constante 'LOAD_PONEYS':

```jsx
// src/constants.js
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";
const LOAD_PONEYS = "LOAD_PONEYS";

export { LOGIN, LOGOUT, LOAD_PONEYS };
```

Agora, vamos ajustar nosso store de poneis para aceitar a ação 'LOAD_PONEYS':

```jsx
// src/reducers/poneys.js
import { LOAD_PONEYS } from "../constants";

const initialState = { list: [], viewDeleted: false };

export default function poneysReducer(state = initialState, action) {
  switch (action.type) {
    case LOAD_PONEYS:
      return {
        ...state,
        list: [...action.data]
      };
    default:
      return state;
  }
}
```

O último passo é fazer com que, no momento de carregamento da tela de listagem, haja a solicitação de carregamento da lista de poneis na aplicação:

```jsx
// src/components/ListarPoneysScreen.js
import { Button, Icon, Left, ListItem, Right, Text } from "native-base";
import PropTypes from "prop-types";
import React from "react";
import { Alert, FlatList, StyleSheet, View } from "react-native";
import { connect } from "react-redux";

// Novidade aqui
import { bindActionCreators } from "redux";
import { loadPoneys } from "../actions";

class ListarPoneysScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  // Novidade aqui
  componentDidMount() {
    this.props.loadPoneys();
  }

  // Código atual omitido
}

// Código atual omitido

// Novidade aqui
const mapDispatchToProps = dispatch =>
  bindActionCreators({ loadPoneys }, dispatch);

ListarPoneysScreen.propTypes = {
  poneys: PropTypes.object,
  profile: PropTypes.object,

  // Novidade aqui
  loadPoneys: PropTypes.func
};

// Código posterior omitido
```

Tudo parece certo, porém quando tentamos executar o código, recebemos a seguinte tela de erro:

![Erro Action Plain](assets/erroaction.png)

Ou seja, uma ação deve retornar um "objeto plano" ou devemos utilizar um middleware customizado, neste caso, vamos utilizar o middleware customizado chamado de `redux-thunk`, primeiro, devemos instalá-lo:

```bash
> npm install --save redux-thunk
```

Agora, vamos ajustar nossa função em `configureStore` para utilizar o redux-thunk:

```jsx
// src/configureStore.js.js
import { applyMiddleware, createStore } from "redux";
import thunk from "redux-thunk";
import rootReducer from "./reducers";

export default function configureStore() {
  let store = createStore(rootReducer, applyMiddleware(thunk));
  return store;
}
```

Nossa action também deve ser ajustada para utilizar uma função ```dispatch```:

```jsx
// src/actions.js
import { loadPoneysAPI } from "./api";
import { LOAD_PONEYS, LOGIN, LOGOUT } from "./constants";

export function loadPoneys() {
  return dispatch => {
    loadPoneysAPI()
      .then(res => {
        dispatch({
          type: LOAD_PONEYS,
          data: res.body
        });
      })
      .catch(error => {
        console.log(error);
      });
  };
}
// Código posterior omitido
```

## Extra: Filtrar poneys por status (Excluido: True/False)

Se fizermos `console.log` da resposta veremos que na verdade vários dos poneis sofreram exclusão lógica e mesmo assim estão sendo exibidos, precisamos filtrá-los na listagem, assim como criar um botão que exiba somente os excluídos ou somente os não excluídos:

```jsx
// src/actions.js
// Código anterior omitido
export function loadPoneys() {
  return dipatch => {
    loadPoneysAPI()
      .then(res => {
        // Novidade aqui
        console.log(">>>>>>>>>>>>>>>>>");
        console.log(res.body);

        dipatch({
          type: LOAD_PONEYS,
          data: res.body
        });
      })
      .catch(error => {
        alert(error.message);
      });
  };
}
// Código posterior omitido
```

A resposta será algo como:

```bash
>>>>>>>>>>>>>>>>>
Array [
  Object {
    "_id": "AUqbjNFR17LqwOac",
    "cor": "Branca",
    "excluido": true,
    "nome": "Epona",
  },
  Object {
    "_id": "cU2BieASCEEIgF8E",
    "cor": "Preta",
    "excluido": true,
    "nome": "Pé de Pano",
  },
  Object {
    "_id": "z7wyKpalWQe7LeJz",
    "cor": "Malhada",
    "excluido": true,
    "nome": "Thor",
  },
  Object {
    "_id": "MIofwryRe6FKpO9G",
    "cor": "Baio",
    "nome": "Trovão",
  },
]
```

Vamos então colocar o botão com a funcionalidade:

```jsx
// src/components/HeaderButtonsComponent.js
// Código anterior omitido
import { bindActionCreators } from "redux";

// Novidade aqui
import { login, logout, toggleViewDeletedPoneys } from "../actions";

class HeaderButtonsComponent extends React.Component {
  // Código omitido

  render() {
    return (
      <View style={styles.headerButtonContainer}>
        {/* Novidade aqui */}
        <Button transparent onPress={this.props.toggleViewDeletedPoneys}>
          <Icon
            style={[styles.headerIconFont, styles.headerIconMargin]}
            name={this.props.poneys.viewDeleted ? "eye-off" : "eye"}
          />
        </Button>

        {this.props.profile.user ? (
          <View style={styles.headerButtonContainer}>
            <Button transparent>
              <Icon
                style={[styles.headerIconFont, styles.headerIconMargin]}
                name="add"
                onPress={() =>
                  Alert.alert("Incluir", "Aqui irá a tela de incluir ponei", [
                    { text: "OK" }
                  ])
                }
              />
            </Button>
            <Button transparent onPress={this.handleLogout}>
              <Image
                style={styles.headerIconMargin}
                source={require("../assets/admin.png")}
              />
            </Button>
          </View>
        ) : (
          <Button transparent onPress={this.openLoginModal}>
            <Icon
              style={[styles.headerIconFont, styles.headerIconMargin]}
              name="contact"
            />
          </Button>
        )}
        {this.renderLoginModal()}
      </View>
    );
  }
}

// Código omitido

HeaderButtonsComponent.propTypes = {
  profile: PropTypes.object,
  poneys: PropTypes.object,

  // Novidade aqui
  toggleViewDeletedPoneys: PropTypes.func
};

const mapStateToProps = state => {
  return {
    profile: state.profile,
    poneys: state.poneys
  };
};

const mapDispatchToProps = dispatch =>
  // Novidade aqui
  bindActionCreators({ login, logout, toggleViewDeletedPoneys }, dispatch);

// Código posterior omitido
```

Agora, temos que criar uma constante para uma nova ação de habilitar e desabilitar a exibição de poneys ocultos, assim como tratar esta ação no reducer e criar uma action correspondente:

```jsx
// src/constants.js
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";
const LOAD_PONEYS = "LOAD_PONEYS";

// Novidade aqui
const TOGGLE_VIEW_DELETED_PONEYS = "TOGGLE_VIEW_DELETED_PONEYS";

// Novidade aqui
export { LOGIN, LOGOUT, LOAD_PONEYS, TOGGLE_VIEW_DELETED_PONEYS };
```

```jsx
// src/reducers/poneys.js
// Novidade aqui
import { LOAD_PONEYS, TOGGLE_VIEW_DELETED_PONEYS } from "../constants";

const initialState = { list: [], viewDeleted: false };

export default function poneysReducer(state = initialState, action) {
  switch (action.type) {
    case LOAD_PONEYS:
      return {
        ...state,
        list: [...action.data]
      };

    // Novidade aqui
    case TOGGLE_VIEW_DELETED_PONEYS:
      return {
        ...state,
        viewDeleted: !state.viewDeleted
      };
    default:
      return state;
  }
}
```

```jsx
// src/actions.js
// Código anterior omitido
import { loadPoneysAPI } from "./api";
import {
  LOAD_PONEYS,
  LOGIN,
  LOGOUT,

  // Novidade aqui
  TOGGLE_VIEW_DELETED_PONEYS
} from "./constants";

// Código omitido

// Novidade aqui
export function toggleViewDeletedPoneys() {
  return {
    type: TOGGLE_VIEW_DELETED_PONEYS
  };
}
// Código posterior omitido
```

E o último passo é responder a esta nova propriedade do nosso estado na view de listagem de poneys:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
class ListarPoneysScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  componentDidMount() {
    this.props.loadPoneys();
  }

  render() {

    // Novidades aqui
    let { poneys } = this.props;

    let listExibicao = poneys.list.filter(
      p => this.props.poneys.viewDeleted === !!p.excluido
    );

    return (
      <View>
        <FlatList

          {/* Novidades aqui */}
          data={listExibicao}

          extraData={this.props.profile}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text
                  {/* Novidades aqui */}
                  style={[styles.item, item.excluido ? styles.tachado : ""]}
                >
                  {item.nome}
                </Text>
              </Left>
              {/* Código omitido */}
            </ListItem>
          )}
          keyExtractor={item => item._id}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  itemContainer: {
    flex: 1,
    flexDirection: "row",
    borderBottomWidth: 1
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
    width: 120
  },

  // Novidades aqui
  tachado: {
    textDecorationLine: "line-through",
    textDecorationStyle: "solid"
  }
});

// Código posterior omitido
```

Mas agora temos um dilema, não seria mais correto ocultar os botões de edição e exclusão de poneis para os poneis que já foram excluídos?

Vamos fazer isso:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
class ListarPoneysScreen extends React.Component {
  // Código omitido

  render() {
    // Código omitido

    return (
      <View>
        <FlatList
          data={listExibicao}
          extraData={this.props.profile}
          renderItem={({ item }) => (
            <ListItem noIndent>
              <Left>
                <Text
                  style={[styles.item, item.excluido ? styles.tachado : ""]}
                >
                  {item.nome}
                </Text>
              </Left>
              <Right>
                {/* Novidades aqui */}
                {this.props.profile.user && !item.excluido && (
                  <View style={{ flexDirection: "row", flex: 1 }}>
                    <Button
                      primary
                      onPress={() =>
                        Alert.alert(
                          "Alterar",
                          "Aqui irá a tela de Alterar ponei",
                          [{ text: "OK" }]
                        )
                      }
                      style={{ marginRight: 10 }}
                    >
                      <Icon name="create" />
                    </Button>
                    <Button
                      danger
                      onPress={() =>
                        Alert.alert(
                          "Excluir",
                          "Aqui exibirá a confirmação de exclusão do ponei",
                          [{ text: "OK" }]
                        )
                      }
                    >
                      <Icon name="trash" />
                    </Button>
                  </View>
                )}
              </Right>
            </ListItem>
          )}
          keyExtractor={item => item._id}
        />
      </View>
    );
  }
}

// Código posterior omitido
```
