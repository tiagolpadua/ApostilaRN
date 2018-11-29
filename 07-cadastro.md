# Criando a tela de Cadastro de Poneys

## Criando rota para a tela

Nossa próxima tarefa é criar um formulário que permita o cadastramento de novos poneis no sistema, vamos criar o componente:

```jsx
// src/components/AdicionarPoneyScreen.js
import { Text } from "native-base";
import React from "react";
import { connect } from "react-redux";

class AdicionarPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return <Text>Tela de Inclusão de Poneis</Text>;
  }
}

const mapStateToProps = () => ({});

const mapDispatchToProps = () => ({});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(AdicionarPoneyScreen);
```

Temos também que adicionar um roteamento para esta tela:

```jsx
// src/CoponeyMobNav.js
// Código anterior omitido
import AdicionarPoneyScreen from "./components/AdicionarPoneyScreen";

const RootStack = createStackNavigator(
  {
    ListarPoneys: {
      screen: ListarPoneysScreen,
      navigationOptions: {
        title: "Lista de Poneys",
        headerRight: <HeaderButtonsComponent />
      }
    },

    // Novidade aqui
    AdicionarPoney: {
      screen: AdicionarPoneyScreen,
      navigationOptions: {
        title: "Adicionar Poney"
      }
    }
  },
  {
    initialRouteName: "ListarPoneys"
  }
);
// Código posterior omitido
```

Agora temos que "navegar" de fato da tela de listagem para a tela que adiciona os poneys pelo evento de click do usuário no botão de inclusão de ponei que fica no componente de cabeçalho:

```jsx
// src/components/HeaderButtonsComponent.js
// Código anterior omitido
render() {
  return (
    <View style={styles.headerButtonContainer}>
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

              {/* Novidade aqui */}
              onPress={() => this.props.navigation.navigate("AdicionarPoney")}
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

// Código omitido


HeaderButtonsComponent.propTypes = {
  profile: PropTypes.object,
  poneys: PropTypes.object,
  toggleViewDeletedPoneys: PropTypes.func,

  // Novidade aqui
  navigation: PropTypes.object
};

// Código posterior omitido
```

Mas este objeto navigation tem que ser disponibilizado para o nosso componente, e isso é feito alterando-se o componente de navegação:

```jsx
// src/CoponeyMobNav.js
// Código anterior omitido
import AdicionarPoneyScreen from "./components/AdicionarPoneyScreen";

const RootStack = createStackNavigator(
  {
    // Novidade aqui
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
    }
  },
  {
    initialRouteName: "ListarPoneys"
  }
);
// Código posterior omitido
```

## Criando o formulário de inclusão de poneis

Agora que já temos um componente pronto para ser nossa tela de inclusão de poneis e o roteamento também já está funcionando, é hora de começarmos a trabalhar no formulário, uma boa estratégia é criar um componente separado para o formulário de modo que ele possa ser reaproveitado para o fluxo de alteração de poneis.

Para facilitar o processo de validação do formulário, assim como o controle dos dados impostados, utilizaremos a biblioteca `redux-form`, logo, o primeiro passo é instalá-la:

```bash
> npm install --save redux-form
```

```jsx
// src/components/MantemPoneyForm.js
import { Button, Text, Item, Label, Input, Picker, Icon } from "native-base";
import React from "react";
import { View } from "react-native";
import { Field, reduxForm } from "redux-form";

const validate = values => {
  const error = {
    nome: "",
    cor: ""
  };
  let nome = values.nome || "";
  let cor = values.cor || "";
  if (nome.length < 3) {
    error.nome = "Muito curto";
  }
  if (nome.length > 8) {
    error.nome = "Máximo de 8 caracteres";
  }
  if (!cor) {
    error.cor = "Cor inválida";
  }
  return error;
};

class MantemPoneyForm extends React.Component {
  renderInput = ({ input, label, meta: { error } }) => {
    var hasError = false;
    if (error !== undefined) {
      hasError = true;
    }
    return (
      <Item floatingLabel error={hasError}>
        <Label>
          {label} {hasError && " - " + error}
        </Label>
        <Input {...input} />
      </Item>
    );
  };

  renderPicker = ({
    input: { onChange, value, ...inputProps },
    ...pickerProps
  }) => {
    return (
      <Item picker>
        <Picker
          selectedValue={value}
          onValueChange={value => onChange(value)}
          {...inputProps}
          {...pickerProps}
          mode="dropdown"
          iosIcon={<Icon name="ios-arrow-down-outline" />}
          style={{ width: undefined }}
          placeholder="Selecione a cor"
          placeholderStyle={{ color: "#bfc6ea" }}
          placeholderIconColor="#007aff"
        >
          <Picker.Item label="Selecione a cor" value="" />
          <Picker.Item label="Amarelo" value="Amarelo" />
          <Picker.Item label="Baio" value="Baio" />
          <Picker.Item label="Branco" value="Branco" />
          <Picker.Item label="Preto" value="Preto" />
        </Picker>
      </Item>
    );
  };

  render() {
    const { invalid, handleSubmit } = this.props;
    return (
      <View style={{ marginTop: 10 }}>
        <Field name="nome" label="Nome" component={this.renderInput} />
        <Field name="cor" label="Cor" component={this.renderPicker} />
        <Button
          disabled={invalid}
          bordered={invalid}
          full
          primary
          style={{ marginBottom: 20 }}
          onPress={handleSubmit}
        >
          <Text>Salvar</Text>
        </Button>
        <Button
          full
          warning
          onPress={() => this.props.navigation.navigate("ListarPoneys")}
        >
          <Text>Cancelar</Text>
        </Button>
      </View>
    );
  }
}

export default reduxForm({
  form: "mantemPonei",
  validate
})(MantemPoneyForm);
```

Para que o `redux-form` funcione, precisamos incluí-lo como um reducer quando criamos a nossa store:

```jsx
// src/reducers/index.js
import { combineReducers } from "redux";
import poneysReducer from "./poneys";
import profileReducer from "./profile";

// Novidade aqui
import { reducer as formReducer } from "redux-form";

const rootReducer = combineReducers({
  profile: profileReducer,
  poneys: poneysReducer,

  // Novidade aqui
  form: formReducer
});

export default rootReducer;
```

Quando incluimos o componente do formulário que criamos, é necessário passar uma função como parâmetro `handleSubmit` que irá receber o evento de submissão juntamente com os dados do ponei que foi cadastrado. Vamos ajustar agora o componente `AdicionaPoneyScreen` para que ele incorpore o formulário de poneis:

```jsx
// src/components/AdicionarPoneyScreen.js
import React from "react";
import { connect } from "react-redux";

// Novidade aqui
import PropTypes from "prop-types";
import MantemPoneyForm from "./MantemPoneyForm";

class AdicionaPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  // Novidade aqui
  handleAddPoney = poney => {
    alert("Ponei adicionado: " + JSON.stringify(poney));
    this.props.navigation.navigate("ListarPoneys");
  };

  // Novidade aqui
  render() {
    return (
      <MantemPoneyForm
        navigation={this.props.navigation}
        onSubmit={this.handleAddPoney}
      />
    );
  }
}

// Novidade aqui
AdicionaPoneyScreen.propTypes = {
  navigation: PropTypes.object
};

// Código posterior omitido
```

## Mapeamento do Serviço Rest para Inclusão de Ponei

Agora que já temos um mapeamento e um formulário funcional para cadastramento de poneis, precisamos nos comunicar com o back-end para que seja feita a persistência dos dados.

Como sempre, vamos criar constantes, chamada de API, ações e alterar nosso reducer incluindo esta funcionalidade:

- Constantes:

```jsx
// src/constants.js
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";
const LOAD_PONEYS = "LOAD_PONEYS";
const TOGGLE_VIEW_DELETED_PONEYS = "TOGGLE_VIEW_DELETED_PONEYS";

// Novidade aqui
const ADD_PONEY = "ADD_PONEY";

// Novidade aqui
export { ADD_PONEY, LOGIN, LOGOUT, LOAD_PONEYS, TOGGLE_VIEW_DELETED_PONEYS };
```

- API:

```jsx
// src/api/index.js
import request from "superagent";

const URI = "https://coponeyapi.herokuapp.com/v1/poneys";

export function loadPoneysAPI() {
  return request.get(URI).set("Accept", "application/json");
}

// Novidade aqui
export function addPoneyAPI(poney) {
  return request.post(URI).send(poney);
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

  // Novidade aqui
  ADD_PONEY
} from "./constants";

// Código omitido

export function addPoney(data) {
  return dipatch => {
    addPoneyAPI(data)
      .then(res => {
        dipatch({
          type: ADD_PONEY,
          data: { ...data, _id: res.body }
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
  // Novidade aqui
  LOAD_PONEYS,
  TOGGLE_VIEW_DELETED_PONEYS,
  ADD_PONEY
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

    // Novidade aqui
    case ADD_PONEY:
      return {
        ...state,
        list: [...state.list, action.data]
      };
    default:
      return state;
  }
}
```

E por fim, vamos ligar a função do componente `AdicionarPoneyScreen` à ação que foi criada:

```jsx
// src/components/AdicionarPoneyScreen.js
// Código anterior omitido
import PropTypes from "prop-types";
import React from "react";
import { connect } from "react-redux";
import MantemPoneyForm from "./MantemPoneyForm";

// Novidade aqui
import { bindActionCreators } from "redux";
import { addPoney } from "../actions";

class AdicionaPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  handleAddPoney = poney => {
    // Novidade aqui
    this.props.addPoney(poney);

    this.props.navigation.navigate("ListarPoneys");
  };

  render() {
    return (
      <MantemPoneyForm
        navigation={this.props.navigation}
        onSubmit={this.handleAddPoney}
      />
    );
  }
}

// Novidade aqui
AdicionaPoneyScreen.propTypes = {
  addPoney: PropTypes.func,
  navigation: PropTypes.object
};

const mapStateToProps = () => ({});

// Novidade aqui
const mapDispatchToProps = dispatch =>
  bindActionCreators(
    {
      addPoney
    },
    dispatch
  );
// Código posterior omitido
```
