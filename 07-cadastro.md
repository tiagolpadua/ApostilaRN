# Criando a tela de Cadastro de Poneys

## Criando rota para a tela

Nossa próxima tarefa é criar um formulário que permita o cadastramento de novos poneis no sistema, vamos criar o componente:

```jsx
// src/components/AdicionarPoneyScreen.js
import { Text } from "native-base";
import React from "react";
import { connect } from "react-redux";

class AdicionaPoneyScreen extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return <Text>Tela de Inclusão de Poneis</Text>;
  }
}

const mapStateToProps = {};

const mapStateToProps = () => ({});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(AdicionaPoneyScreen);
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
