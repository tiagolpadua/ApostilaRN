# Criando a tela de Alteração de Poneys

## Criando a rota para a tela

Basicamente iremos repetir os passos que fizemos quando criamos a tela de cadastramento de poneis, primeiro, criaremos um componente que conterá o formulário que será reaproveitado:

```jsx
// src/components/AtualizarPoneyScreen.js
import PropTypes from "prop-types";
import React from "react";
import { connect } from "react-redux";
import MantemPoneyForm from "./MantemPoneyForm";

class AtualizarPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  handleUpdatePoney = poney => {
    alert("Atualizar ponei: " + JSON.stringify(poney));
    this.props.navigation.navigate("ListarPoneys");
  };

  render() {
    return (
      <MantemPoneyForm
        initialValues={this.props.navigation.getParam("poney")}
        navigation={this.props.navigation}
        onSubmit={this.handleUpdatePoney}
      />
    );
  }
}

AtualizarPoneyScreen.propTypes = {
  updatePoney: PropTypes.func,
  navigation: PropTypes.object
};

const mapStateToProps = () => ({});

const mapDispatchToProps = () => ({});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(AtualizarPoneyScreen);
```

Agora, incluimos uma rota nova no componente de navegação:

```jsx
// src/CoponeyMobNav.js
// Código anterior omitido

// Novidade aqui
import AtualizarPoneyScreen from "./components/AtualizarPoneyScreen";

const RootStack = createStackNavigator(
  {
    ListarPoneys: {
      screen: ListarPoneysScreen,
      navigationOptions: ({ navigation }) => ({
        title: "Lista de Poneys",
        headerRight: <HeaderButtonsComponent navigation={navigation} />
      })
    },
    AdicionarPoney: {
      screen: AdicionarPoneyScreen,
      navigationOptions: {
        title: "Adicionar Poney"
      }
    },

    // Novidade aqui
    AtualizarPoney: {
      screen: AtualizarPoneyScreen,
      navigationOptions: {
        title: "Atualizar Poney"
      }
    }
  },
  {
    initialRouteName: "ListarPoneys"
  }
);

// Código posterior omitido
```

Por fim, ajustamos a tela de listagem de poneis para que o botão de edição navege para a tela que foi criada, passando como parâmetro o ponei que será atualizado:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
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
                {this.props.profile.user && !item.excluido && (
                  <View style={{ flexDirection: "row", flex: 1 }}>
                    <Button
                      primary

                      {/* Novidade aqui */}
                      onPress={() =>
                        this.props.navigation.navigate("AtualizarPoney", {
                          poney: item
                        })
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

// Código omitido

ListarPoneysScreen.propTypes = {
  poneys: PropTypes.object,
  profile: PropTypes.object,

  // Novidades aqui
  loadPoneys: PropTypes.func,
  navigation: PropTypes.object
};

// Código posterior omitido
```

## Mapeamento do Serviço Rest para Atualização do Poney

Novamente chega a hora de nos comunicarmos com o back-end para que seja feita a persistência dos dados.

Como sempre, vamos criar constantes, chamada de API, ações e alterar nosso reducer incluindo esta nova funcionalidade:

- Constantes:

```jsx
// src/constants.js
const ADD_PONEY = "ADD_PONEY";
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";
const LOAD_PONEYS = "LOAD_PONEYS";
const TOGGLE_VIEW_DELETED_PONEYS = "TOGGLE_VIEW_DELETED_PONEYS";

// Novidade aqui
const UPDATE_PONEY = "UPDATE_PONEY";

export {
  ADD_PONEY,
  LOGIN,
  LOGOUT,
  LOAD_PONEYS,
  TOGGLE_VIEW_DELETED_PONEYS,
  // Novidade aqui
  UPDATE_PONEY
};
```

- API:

```jsx
// src/api/index.js
import request from "superagent";

const URI = "https://coponeyapi.herokuapp.com/v1/poneys";

export function loadPoneysAPI() {
  return request.get(URI).set("Accept", "application/json");
}

export function addPoneyAPI(poney) {
  return request.post(URI).send(poney);
}

// Novidade aqui
export function updatePoneyAPI(poney) {
  return request.put(URI + "/" + poney._id).send(poney);
}
```

- Ações:

```jsx
// src/actions.js
// Novidade aqui
import { loadPoneysAPI, addPoneyAPI } from "./api";

import {
  LOAD_PONEYS,
  LOGIN,
  LOGOUT,
  TOGGLE_VIEW_DELETED_PONEYS,
  ADD_PONEY,

  // Novidade aqui
  UPDATE_PONEY
} from "./constants";

// Código omitido

export function updatePoney(data) {
  return dipatch => {
    updatePoneyAPI(data)
      .then(() => {
        dipatch({
          type: UPDATE_PONEY,
          data
        });
      })
      .catch(error => {
        alert(error.message);
      });
  };
}

// Código posterior omitido
```

- Reducer:

```jsx
// src/reducers/poneys.js
// Código anterior omitido
import {
  LOAD_PONEYS,
  TOGGLE_VIEW_DELETED_PONEYS,
  ADD_PONEY,

  // Novidade aqui
  UPDATE_PONEY
} from "../constants";

const initialState = { list: [], viewDeleted: false };

export default function poneysReducer(state = initialState, action) {
  switch (action.type) {
    case LOAD_PONEYS:
      return {
        ...state,
        list: [...action.data]
      };
    case TOGGLE_VIEW_DELETED_PONEYS:
      return {
        ...state,
        viewDeleted: !state.viewDeleted
      };

    case ADD_PONEY:
      return {
        ...state,
        list: [...state.list, action.data]
      };

    // Novidade aqui
    case UPDATE_PONEY:
      return {
        ...state,
        list: state.list.map(p =>
          p._id === action.data._id ? { ...action.data } : p
        )
      };
    default:
      return state;
  }
}
```

E por fim, vamos ligar a função do componente `AtualizarPoneyScreen` à ação que foi criada:

```jsx
// src/components/AtualizarPoneyScreen.js
// Código anterior omitido

// Novidades aqui
import { bindActionCreators } from "redux";
import { updatePoney } from "../actions";
import MantemPoneyForm from "./MantemPoneyForm";

class AtualizarPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  handleUpdatePoney = poney => {
    // Novidades aqui
    this.props.updatePoney(poney);

    this.props.navigation.navigate("ListarPoneys");
  };

  // Código omitido
}

// Código omitido

const mapDispatchToProps = dispatch =>
  bindActionCreators(
    {
      updatePoney
    },
    dispatch
  );

// Código posterior omitido
```
