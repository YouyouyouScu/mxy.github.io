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
  **action**: action是将数据从应用传到store的有效负载，必须有一个type,如`{ type: 'ADD_TODO', text: 'Go to swimming pool' }`  
    action为了方便使用和书写，可以定义其creator  
    ```
    function addTodo(text) {
      return {
        type: ADD_TODO,
        text
      };
    }
    ```
  **reducer**: reducer是根据action.type识别state怎么被修改的处理函数，他是纯函数，定义：`type Reducer<S, A> = (state: S, action: A) => S; ` 
  reducer有两个参数，state和action, 返回值为新的state  
  CombineReducer的函数(详细解析看引用):
  项目复杂以后，需要拆分reducer单独维护，CombineReducer负责将多个reducer合并成一个reducer  
  最终，state 对象的结构会是这样的:  
  ```
    {
      reducer1: ...
      reducer2: ...
    }
  ```
+ useState、useReducer
  useState和useReducer都是服务于函数组件，使函数组件拥有自己独立的state和reducer来保存和更新组件的私有状态
+ 中间件
  + redux-thunk
  + Redux-Saga  
  Saga中文文档：https://redux-saga-in-chinese.js.org/
  Saga用于拦截异步action, 并且用同步的写法去书写异步处理操作（结合Generator），可读性更加高。  
  本例中使用上文的react context来传递状态  
  场景说明：
  异步修改页面中的count元素
  
    1. 我们得有api去修改count, 代码如下：  
    ```
    // api.js

    function getUserCount() {
      // 使用promise和settimeout去模拟一个异步请求
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(2000);
        }, 1000)
      })
    }

    export { getUserCount }
    ```
    2. 我们需要定义修改state的action, 代码如下：  
    ```
    // actions.js

    import { createAction } from 'redux-actions'
    // 我们有两个action_type，但是只用createAction创建了一个，因为START_CHANGE_COUNT是saga的action, 不会对数据造成实际的操作，可以只定义类型
    const ACTION_TYPE = {
      START_CHANGE_COUNT: 'startChangeCount',
      CHANGE_COUNT: 'changeCount',
    }
    
    const changeCount = createAction(ACTION_TYPE.CHANGE_COUNT);

    const actionsCreator = {
      changeCount
    }

    export { ACTION_TYPE, actionsCreator }
    ```
    3. 创建reducer, 代码如下：  
    ```
    import { ACTION_TYPE } from "./actions.js";

    const initState = {
      count: 0,
      name: 'zhangsan'
    }

    export const reducer = (state = initState, {type, payload}) => {
      switch (type) {
        // 这边我们并没有处理START_CHANGE_COUNT这个类型，而是默认return state，因为saga的action可以理解为中间桥梁，不改变数据
        // 只有当异步操作结束以后，才会调用内部的action去修改数据
        case ACTION_TYPE.CHANGE_COUNT: {
          return {
            ...state,
            count: payload,
          }
        }
        default:
          return state;
      }
    }

    export { ACTION_TYPE }
    ```
    前三步，我们都是和saga没有任何关系，只是按照常规定义api, action和reducer.  
    4. 这一步，我们开始编写saga的监听和拦截逻辑，代码如下：  
    ```
    // sagaEffects.js

    import { ACTION_TYPE, actionsCreator } from "./actions.js";
    import { getUserCount } from "./api.js";
    // api的使用可以去官网查看，这边不赘述
    import { all, call, put, takeEvery } from 'redux-saga/effects'
    
    //在这个函数中，我们调用异步接口去获取Count, put函数会帮我们去调用redux的dispatch函数，真正的修改state的数据
    function* fecthCount() {
      const count = yield call(getUserCount);
      yield put(actionsCreator.changeCount(count));
    }
    // 这个函数，监听saga的action: START_CHANGE_COUNT, 如果用户触发START_CHANGE_COUNT, 就执行fecthCount函数
    function* watcherCount() {
      yield takeEvery(ACTION_TYPE.START_CHANGE_COUNT, fecthCount);
    }
    // 这个函数只是用all把我们的监听函数封装一下，如果有多个saga action，除了用all, 也可以直接写多个 takeEvery
    export function* mySaga() {
      yield all([watcherCount()]);
      // 直接使用下面写法也是可以的
      // yield takeEvery(ACTION_TYPE.START_CHANGE_COUNT, fecthCount);
    }
    ```
    4. 上面我们已经生成了saga的action逻辑，那么就在组件中触发saga的action, 代码如下：
    ```
    // App.js
    import React, { useContext } from 'react';
    import AppContext from "./AppContext";
    import { ACTION_TYPE } from "../saga";

    export default function App() {
      const [state, dispatch] = useContext(AppContext);

      return (
        <div>
          {state.name} ===========
          {state.count}
          <button type='primary' onClick={() => dispatch({ type: ACTION_TYPE.START_CHANGE_COUNT})}> change State </button>
        </div>
      );
    }
    
    // AppContext.js
    import React from "react";
    // Page content, how to use:https://reactjs.org/docs/hooks-reference.html#usecontext
    const AppContext = React.createContext();
    export default AppContext;
    ```
    5. 最后我们在入口文件中引入saga
    ```
    // index.js
    // saga test
    import React from 'react';
    import ReactDOM from 'react-dom';
    import App from "./sagatest/App.js";
    // 引入applyMiddleware，用来将saga放入store的中间件中
    import { createStore, applyMiddleware } from 'redux'
    // 引入createSagaMiddleware， 用来创建全局saga实例
    import createSagaMiddleware from 'redux-saga'
    // 将自己写的reducer, mySaga引入
    import { reducer, mySaga } from "./saga";
    import AppContext from "./sagatest/AppContext.js";
    // 创建saga实例
    const sagaMiddleware = createSagaMiddleware()
    // 创建store时，引入saga实例
    const AppStore = createStore(
      reducer,
      applyMiddleware(sagaMiddleware)
    );
    // 开启全局监听saga的action
    sagaMiddleware.run(mySaga)

    const render = () => {
      ReactDOM.render(
        // 这一步，将AppStore作为上下文参数传给子组件，App使用useContext获取
        <AppContext.Provider value={[AppStore.getState(), AppStore.dispatch]}>
          <App />
        </AppContext.Provider>,
        document.getElementById('root')
      );
    };
    //渲染
    render();

    AppStore.subscribe(render);
    ```
    代码主要目录结构如下:  
    > saga  
      >> actions.js  
      >> api.js  
      >> index.js  
      >> reducer.js  
      >> sagaEffects.js  
    > sagatest  
      >> App.js  
      >> AppContext.js  
    index.js  
    saga > index.js 代码如下： 
    ```
    export * from './reducer'
    export * from './sagaEffects'
    ```
  参考：  
  > 1.http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html (一，二，三篇)
  > 2.https://www.cnblogs.com/wy1935/p/7109701.html
  > 3.http://www.ruanyifeng.com/blog/2015/04/generator.html
  
  
