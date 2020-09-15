# Context

### step1 (createContext)

first you have to create a folder called `context` and then inside that folder create the context file which you want.

for example, I want to control the theme of the page so I'm going to create the ThemeContext.

`ThemeContext.js`

```js
import React, { createContext } from "react";

export const ThemeContext = createContext();

export class ThemeProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      isDarkMode: false,
    };
  }

  render() {
    return (
      <ThemeContext.Provider value={{ ...this.state, anotherState: true }}>
        {this.props.children}
      </ThemeContext.Provider>
    );
  }
}
```

### step2 (Provide Context)

as we created a component called ThemeProvider, here we have wrap the component which we want to use the `states` of the ThemeProvider

`App.js`

```js
import React from "react";
import "./App.css";

// context
import { ThemeProvider } from "./context/ThemeContext";

// components
import PageContent from "./components/pageContent/PageContent.component.jsx";
import Navbar from "./components/navbar/Navbar.component.jsx";
import Form from "./components/form/Form.component.jsx";

function App() {
  return (
    <ThemeProvider>
      <PageContent>
        <Navbar />
        <Form />
      </PageContent>
    </ThemeProvider>
  );
}

export default App;
```

### step2 (Consume context)

#### class based components

`MyClass.js`:

```js
import React, {Component} from 'react';
import {ThemeContext} from './context/ThemeContext';

class MyClass extends Component {
  static contextType = ThemeContext;
  render() {
    const { isDarkMode } = this.context;
    return (
      <AppBar color={isDarkMode ? 'default' : 'primary'}>
        this is MyClass Component
      </AppBar>
    )
  }
}
```

Here: 

```js
console.log(this.context) // This will return you the value object which you have assigned in <ThemeContext.Provider value={{ ...this.state, anotherState: true }}></ThemeContext.Provider>
```

**If you want to change the styles based off of the context state, Remember that you should write the styles inside the component**

for example:

`PageContent.js`:

```js
import React, { Component } from 'react';
import { ThemeContext } from './context/ThemeContext';

class PageContent extends Component {
  static contextType = ThemeContext;
  render() {
    const { isDarkMode } = this.context;
    const styles = {
      width: '100vw',
      height: '100vh',
      backgroundColor: isDarkMode ? 'black' : 'white'
    }
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
}
```

**Adding Method or a function into context (Updatig the context)**
for example I want to have a function called `toggleTheme` to toggle the `isDarkMode` from false to true and true to false

`ThemeContext.js`

```js
import React, { createContext } from "react";

export const ThemeContext = createContext();

export class ThemeProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      isDarkMode: false,
    };

    this.toggleTheme = this.toggleTheme.bind(this);
  }

  toggleTheme() {
    this.setState({
      isDarkMode: !this.state.isDarkMode,
    });
  }

  render() {
    return (
      <ThemeContext.Provider
        value={{ ...this.state, toggleTheme: this.toggleTheme }}
      >
        {this.props.children}
      </ThemeContext.Provider>
    );
  }
}
```

And the to use it inside the compoent, it is exatly the same before

`MyClass.js`:

```js
import React, {Component} from 'react';
import {ThemeContext} from './context/ThemeContext';

class MyClass extends Component {
  static contextType = ThemeContext;
  render() {
    const { isDarkMode, toggleTheme } = this.context;
    return (
      <AppBar color={isDarkMode ? 'default' : 'primary'}>
        <Switch onChange={toggleTheme} />
      </AppBar>
    )
  }
}
```


Another Example for class based component:

**LanguageContext**

`LanguageContext.js`

```js
import React, { Component, createContext } from 'react';

export const LanguageContext = createContext();

export class LanguageProvider extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      language: 'english'
    }
    
    this.changeLanguage = this.changeLanguage.bind(this);
  }
  
  changeLanguage(e) {
    this.setState({
      language: e.target.value
    })
  }
  
  render() {
    return (
      <LanguageContext.Provider value={{ ...this.state, changeLanguage: this.changeLanguage }}>
        {this.props.children}
      </LanguageContext.Provider>
    )
  }
}
```

Then you have to provide your components with the Context Provider.

Then You can consume it inside your components:

`MyClass.js`

```js
import React, { Component } from 'react';
import { LanguageContext } from './context/LanguageContext';

const content = [
  english: {
     email: 'Email',
     signIn: 'Sign in'
  },
  
  french: {
    emaiL: 'Addresee Electronico',
    signIn: 'Se Connecter'
  },
  
  spanish: {
    email: 'Correo Electronico',
    signIn: 'Registrarse'
  }
]

class MyClass extends Component {
  static contextType = LanguageContext;
  const { language, changeLanguage } = LanguageContext;
  const { email, signIn } = content[language]
  render() {
    return (
      <div>
        <Select value={language} onChange={changeLanguage}>
          <MenuItem value='english'>English</MenuItem>
          <MenuItem value='french'>French</MenuItem>
          <MenuItem value='spanish'>Spanish</MenuItem>
        </Select>
        {email}
        {signIn}
      </div>
    )
  }
}
```

### How to consume several contexts inside one component

Look at this componet:

`Navbar.js`

```js
import React, { Component } from 'react';
import { ThemeContext } from './context/ThemeContext';

class Navbar extends Component {
  static contextType = ThemeContext;
  render() {
    const { isDarkMode } = this.context;
    const styles = {
      backgroundColor: isDarkMode ? 'black' : 'white'
    }
    return (
      <div style={styles}>aaa</div>
    )
  }
}

export default Navbar;
```

Now as you can see I have used the `ThemeContext` in this component and I have defined it using `static contextType = ThemeContext`;

Now I want to use another context called `LanguageContext` in this component.

But there is a problem I can't consume the **`second` context** exactly the same way I consumed the first one.

**SOLUTION**

You have to create a `higher order component` of the second `context`.

**How?**

You have to define and export the `LanguageContext.Consumer` from the `LanguageContext` component

`LanguageContext.js`

```js
import React, { createContext, Component } from 'react';

export const LanguageContext = createContext();

export class LanguageProvider extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      language: 'english'
    }
  }
  
  changeLanguage(e) {
    this.setState({ language: e.target.value })
  }
  
  render() {
    return (
      <LanguageContext.Provider value={{ ...this.state, changeLanguage: this.changeLanguage }}
        {this.props.children}
      </LanguageContext.Provider>
    )
  }
}

// This is the LanguageContext Consumer Higher Order Component
export const withLanguageContext = Component => props => (
  <LanguageContext.Consumer>
    {value => <Component languageContext={value} {...props} />}
  </LanguageContext.Consumer>
)
```

after that we defined the `withLanguageContext` higher order component, then we can use it inside our `Navbar` component as the second context.

`Navbar.js`

```js
import React, { Component } from 'react';
import { ThemeContext } from './context/ThemeContext';
import { withLanguageContext } from './context/LanguageContext';

const content = {
  english: {
    email: 'Email'
  },
  
  french: {
    emaiL 'Addresse Electronico'
  }
}

class Navbar extends Component {
  static contextType = ThemeContext;
  
  render() {
    const { isDarkMode } = this.context;
    const { language, changeLanguage } = this.props.languageContext;  // notice that I wrote languageContext not withLanguageContext
    const { email } = content[language]
    const styles = {
      backgroundColor: isDarkMode ? 'black' : 'white'
    }
    return (
      <div style={styles}>{email}</div>
    )
  }
}

export default withLanguageContext(Navbar);
```

Congratulations You completed the how to use `context` inside `class based components`.


### Context API with Hooks

Here we have a hook called `useContext` which we can use to consume a `context`

```js
const value = useContext(myContext);
```

we will use this instead of `static contextType = myContext`

**what's nice about this, is that we can use more than one context inside a component**
**So if we need to use 2 contexts we just need to put two lines**
**Also, you have to remember you have to pass the entire context not just the consumer**

Example (Here I have used one context inside the component):

```js
import React, { useState, useContext } from 'react';
import { LanguageContext } from "../../context/LanguageContext.js";

const content = {
  english: {
    emailWord: "Email",
    userNameWord: "UserName",
    passwordWord: "Password",
    signInWord: "Sign in",
    loginWord: "Login",
  },

  french: {
    emailWord: "Addresse Electronico",
    userNameWord: "Nom d'utilisateur",
    passwordWord: "mot de passe",
    signInWord: "se connecter",
    loginWord: "s'identifier",
  },

  spanish: {
    emailWord: "correo Electronico",
    userNameWord: "nombre de usuario",
    passwordWord: "contraseña",
    signInWord: "registrarse",
    loginWord: "iniciar sesión",
  },
};

const Form = (props) => {
  const { language, changeLanguage } = useContext(LanguageContext);
  const [email, setEmail] = useState("");
  const [userName, setUserName] = useState("");
  const [password, setPassword] = useState("");
  const {
    emailWord,
    userNameWord,
    passwordWord,
    signInWord,
    loginWord,
  } = content[language];
  const { classes } = props;

  console.log(emailWord);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(language, email, userName, password);
  };

  return (
    <main className={classes.main}>
      <Paper className={classes.paper} elevation={3}>
        <Avatar className={classes.icon}>
          <AssignmentIndIcon />
        </Avatar>
        <Typography variant="h6">{signInWord}</Typography>
        <form className={classes.form} onSubmit={handleSubmit}>
          <FormControl className={classes.selectInput}>
            <InputLabel htmlFor="select language">Select Language</InputLabel>
            <Select
              labelId="select language"
              id="demo-simple-select-helper"
              value={language}
              onChange={changeLanguage}
            >
              <MenuItem value="english">English</MenuItem>
              <MenuItem value="french">French</MenuItem>
              <MenuItem value="spanish">Spanish</MenuItem>
            </Select>
          </FormControl>
          <FormControl className={classes.Input}>
            <InputLabel htmlFor="email">{emailWord}</InputLabel>
            <Input
              type="email"
              labelId="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
            />
          </FormControl>
          <FormControl className={classes.Input}>
            <InputLabel htmlFor="userName">{userNameWord}</InputLabel>
            <Input
              type="text"
              labelId="userName"
              value={userName}
              onChange={(e) => setUserName(e.target.value)}
            />
          </FormControl>
          <FormControl className={classes.Input}>
            <InputLabel htmlFor="Password">{passwordWord}</InputLabel>
            <Input
              type="password"
              labelId="Password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
            />
          </FormControl>
          <Button
            type="submit"
            children={loginWord}
            color="primary"
            variant="contained"
            className={classes.loginButton}
          />
        </form>
      </Paper>
    </main>
  );
};

export default withStyles(styles)(Form);
```

Example: (Here I have used 2 contexts inside one component)

```js
import React, { useState, useContext } from 'react';
import { LanguageContext } from './context/LanguageContext.js';
import { ThemeContext } from './context/ThemeContext.js'

const title = {
  english: {
    word: "EN",
  },

  french: {
    word: "FR",
  },

  spanish: {
    word: "SP",
  },
};

const Navbar = () => {
  const classes = useStyles();
  const { isDarkMode, toggleTheme } = useContext(ThemeContext);
  const { language } = useContext(LanguageContext);
  const { word } = title[language];

  return (
    <AppBar position="static" color={isDarkMode ? "default" : "primary"}>
      <Toolbar>
        <IconButton
          edge="start"
          className={classes.menuButton}
          color="inherit"
          aria-label="menu"
        >
          <Switch
            checked={isDarkMode}
            onChange={toggleTheme}
            color="secondary"
            inputProps={{ "aria-label": "secondary checkbox" }}
          />
        </IconButton>
        <Typography variant="h6" className={classes.title}>
          {word}
        </Typography>
      </Toolbar>
    </AppBar>
  );
};

export default Navbar;
```

**As you can see using functional components `Hooks` and `useContext` is much easier and cleaner than using class based components**


### Rewriting the `ContextProvider` with `Hooks`

until now, we've written the `ContextProvider` components using class based components, but now we want to write it using `Hooks`.

example: 

**LanguageProvider (class based):**

```js
import React, { createContext, Component } from "react";

export const LanguageContext = createContext();

export class LanguageProvider extends Component {
  constructor(props) {
    super(props);

    this.state = {
      language: "english",
    };

    this.changeLanguage = this.changeLanguage.bind(this);
  }

  changeLanguage(e) {
    this.setState({
      language: e.target.value,
    });
  }

  render() {
    return (
      <LanguageContext.Provider
        value={{ ...this.state, changeLanguage: this.changeLanguage }}
      >
        {this.props.children}
      </LanguageContext.Provider>
    );
  }
}
```

NOWWWWWWWWWWWW

**LanguageProvider (Hooks)**

```js
import React, { createContext, useState } from 'react';

export const LanguageContext = createContext();

export function LanguageProvider(props) {
  const [language, setLanguage] = useState('english');
  const changeLanguage = e => setLanguage(e.target.value);
  
  return (
    <LanguageContext.Provider value={{ language, changeLanguage }}>
      {props.children}
    </LanguageContext.Provider>
  )
}
```

**As you can see I have converted the `LanguageProvider` to functional component and I used `Hooks` that is much better and cleaner**.


`until now we've learned the basics of context now we want to build a todoApp using **context**`


### Important tip

`type error reducer is not a function`

**always remember you have to put `reducer` before `state` inside `context initializing state`**

CORRECT:
```js
const [eventState, EventDispatch] = useReducer(reducer, state)
```

IN_CORRECT:
```js
const [eventState, EventDispatch] = useReducer(state, reducer)
```
