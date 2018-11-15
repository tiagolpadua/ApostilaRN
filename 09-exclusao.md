# Exclusão lógica de Poneys

## Modal para confirmar a exclusão

Já avançamos muito com nosso sistema, nossa próxima funcionalidade a ser implementada é a exclusão lógica de poneis.

Como é de praxe, o botão de exclusão irá solicitar uma confirmação ao usuário antes de proceder a exclusão de fato, vamos implementar esse recurso:

```jsx
// src/components/ListarPoneysScreen.js
// Código anterior omitido
class ListarPoneysScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  // Novidades aqui
  handleDeletePoney = poney => {
    Alert.alert("Exclusão", `Confirma a exclusão do poney ${poney.nome}?`, [
      { text: "Sim", onPress: () => alert("Ponei excluído: " + poney.nome) },
      { text: "Não", style: "cancel" }
    ]);
  };

  componentDidMount() {
    this.props.loadPoneys();
  }

  render() {
    let { poneys } = this.props;

    let listExibicao = poneys.list.filter(
      p => this.props.poneys.viewDeleted === !!p.excluido
    );

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
                      onPress={() =>
                        this.props.navigation.navigate("AtualizarPoney", {
                          poney: item
                        })
                      }
                      style={{ marginRight: 10 }}
                    >
                      <Icon name="create" />
                    </Button>

                    {/* Novidades aqui */}
                    <Button danger onPress={() => this.handleDeletePoney(item)}>
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

## Mapeamento do Serviço Rest para Exclusão do Poney

E mais uma vez precisamos nos comunicarmos com o back-end para que seja feita a exclusão dos poneis no banco de dados.

Como sempre, vamos criar constantes, chamada de API, ações e alterar nosso reducer incluindo esta nova funcionalidade:

- Constantes:

```jsx
// src/constants.js
const ADD_PONEY = "ADD_PONEY";
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";
const LOAD_PONEYS = "LOAD_PONEYS";
const TOGGLE_VIEW_DELETED_PONEYS = "TOGGLE_VIEW_DELETED_PONEYS";
const UPDATE_PONEY = "UPDATE_PONEY";

// Novidade aqui
const DELETE_PONEY = "DELETE_PONEY";

export {
  ADD_PONEY,
  LOGIN,
  LOGOUT,
  LOAD_PONEYS,
  TOGGLE_VIEW_DELETED_PONEYS,
  UPDATE_PONEY,
  // Novidade aqui
  DELETE_PONEY
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

export function updatePoneyAPI(poney) {
  return request.put(URI + "/" + poney._id).send(poney);
}

// Novidade aqui
export function deletePoneyAPI(id) {
  return request.delete(URI + "/exclusaologica/" + id);
}
```

- Ações:

```jsx
// src/actions.js
import {
  addPoneyAPI,
  deletePoneyAPI,
  loadPoneysAPI,

  // Novidade aqui
  updatePoneyAPI
} from "./api";

import {
  LOAD_PONEYS,
  LOGIN,
  LOGOUT,
  TOGGLE_VIEW_DELETED_PONEYS,
  ADD_PONEY,
  UPDATE_PONEY,

  // Novidade aqui
  DELETE_PONEY
} from "./constants";

// Código omitido

export function deletePoney(id) {
  return dipatch => {
    deletePoneyAPI(id)
      .then(() => {
        dipatch({
          type: DELETE_PONEY,
          data: id
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
  UPDATE_PONEY,

  // Novidade aqui
  DELETE_PONEY
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

    case UPDATE_PONEY:
      return {
        ...state,
        list: state.list.map(p =>
          p._id === action.data._id ? { ...action.data } : p
        )
      };

    // Novidade aqui
    case DELETE_PONEY:
      return {
        ...state,
        list: state.list.map(p =>
          p._id === action.data ? { ...p, excluido: true } : p
        )
      };
    default:
      return state;
  }
}
```

E por fim, vamos ligar a função do componente `ListarPoneyScreen` à ação que foi criada:

```jsx
// src/components/ListarPoneyScreen.js
// Código anterior omitido
// Novidade aqui
import { loadPoneys, deletePoney } from "../actions";

class ListarPoneysScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  handleDeletePoney = poney => {
    Alert.alert("Exclusão", `Confirma a exclusão do poney ${poney.nome}?`, [
      // Novidade aqui
      { text: "Sim", onPress: () => this.props.deletePoney(poney._id) },
      { text: "Não", style: "cancel" }
    ]);
  };

  // Código omitido
}

// Novidade aqui
const mapDispatchToProps = dispatch =>
  bindActionCreators({ loadPoneys, deletePoney }, dispatch);

ListarPoneysScreen.propTypes = {
  poneys: PropTypes.object,
  profile: PropTypes.object,
  loadPoneys: PropTypes.func,
  navigation: PropTypes.object,

  // Novidade aqui
  deletePoney: PropTypes.func
};
// Código posterior omitido
```
