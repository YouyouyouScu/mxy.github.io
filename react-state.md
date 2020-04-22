## state
+ 组件内部state
  + 普通组件  
    **定义和取值**  
    ```
    import React from 'react';
    
    class stateComponent extends React.Component {
      constructor() {
        super();

        this.state = {
          name: "hello World React!"
        }
      }

      changeState() {
        this.setState({
          name: `${this.state.name}1`
        });
      }

      render() {
        return (
          <div>
            {this.state.name}
            <button type='primary' onClick={() => this.changeState()}> change State </button>
          </div>
        );
      }
    }
    ``` 
    
  + 函数组件
    函数组件中没有state只有props，但是后来加上useState，可以使函数组件和普通组件一样有内部的state  
    **定义和取值**  
    ```
    import React, {useState} from 'react';

    function FunComponent() {
      const [name, setName] = useState("hello World React!");
      return (
        <div>
          {name}
          <button type='primary' onClick={() => setName(`${name}fun`)}> change State </button>
        </div>
      )
    }

    export default FunComponent;
    ```
  
+ 全局state  
  + 全局state的创建和使用
    + redux
      ```
      //index.js
      import React from 'react';
      import ReactDOM from 'react-dom';
      import App from "./App";
      import { createStore } from 'redux';
      //创建store
      const AppStore = createStore((state = { name: "hello World React state!", other: "other info" }, action) => {
        switch (action.type) {
          case 'aaa':
            return {
              ...state,
              name: action.payload
            };
          case 'bbb':
            return {
              ...state,
              other: action.payload
            }
          default:
            return state;
        }
      });

      const render = () => {
        //通过props传递全局state到子组件（显然对多层组件传值不够友好）
        ReactDOM.render(
          <React.StrictMode>
            <App value={AppStore.getState()} 
              changeName={() => { AppStore.dispatch({ type: 'aaa', payload: `${AppStore.getState().name}1` })}} />
          </React.StrictMode>,
          document.getElementById('root')
        );
      };    
      render();
      //监听state变化，变化时重新渲染
      AppStore.subscribe(render);
      ```
      ```
      //App.js
      import React from "react";
      function App({ value, changeName }) {
        return (
          <div>
            name is : {value.name}
            <button type='primary' onClick={() => changeName('new Name')}> change State </button>
          </div>
        );
      }

      export default App;
      ```
      使用provider修改传值方式, 在子组件中定义mapStateToPropsmap和DispatchToProps，并且用connect包裹组件创建一个高阶组件
      ```
      //index.js
      import React from 'react';
      import ReactDOM from 'react-dom';
      import App from "./globalstatedemo/App";
      import { Provider } from 'react-redux'

      import { createStore } from 'redux';
      const AppStore = createStore((state = { name: "hello World React state!", other: "other info" }, action) => {
        switch (action.type) {
          case 'aaa':
            return {
              ...state,
              name: action.payload
            };
          case 'bbb':
            return {
              ...state,
              other: action.payload
            }
          default:
            return state;
        }
      });

      const render = () => {
        ReactDOM.render(
          <Provider store={AppStore}>
            <App />
          </Provider>,
          document.getElementById('root')
        );
      };

      render();

      AppStore.subscribe(render);
      ```
      ```
      //App.js
      import React from "react";
      import { connect } from "react-redux";
      //普通组件写法
      // class App extends React.Component {
      //   render() {
      //     const { name, changeName} = this.props;
      //     return (
      //       <div>
      //         name is : {name}
      //         <button type='primary' onClick={() => changeName('new Name')}> change State </button>
      //       </div>
      //     );
      //   }
      // }
      //函数组件写法
      function App({ name, changeName}) {
        return (
          <div>
            name is : {name}
            <button type='primary' onClick={() => changeName('new Name')}> change State </button>
          </div>
        );
      }

      const mapSateToProps = store => ({
        name: store.name
      });

      const mapDispatchToProps = dispatch => ({
        changeName: name => dispatch({ type: 'aaa', payload: name})
      });

      export default connect(mapSateToProps, mapDispatchToProps)(App);
      ```
    + react context
      react context使得状态管理不依赖于redux, 16.3版本以后支持，下面为核心示例代码
      ```
      import React from "react";
      const AppContext = React.createContext();
      //render部分
      return (
        <AppContext.Provider value={/* your data*/}>
          <App />
        </AppContext.Provider>
      );
      
      //子组件部分获取数据
      //1.使用useContext(代码更加简洁)
      const /* your data */ = useContext(AppContext);
      //2.使用consumer
      return (
          //Consumer容器,可以拿到上文传递下来的name属性,并可以展示对应的值
          <Consumer>
              {( /* your data*/ ) =>
                  <div>
                  /*
                    your code
                  */
                 </div>
              }
          </Consumer>
      );
      ```
      
+ state修改
  + reducer和action
