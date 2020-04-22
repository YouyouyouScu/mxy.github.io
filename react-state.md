## state
+ 组件内部state
  + 普通组件  
    **定义和取值**  
    ```
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
  
+ 全局state
  + 创建全局state
  + 子组件使用全局state
+ state修改
  + reducer和action
