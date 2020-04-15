### Vue中用computed来表示计算属性，那React中有计算属性吗？若有怎么表示？
答：有的，书写风格有React Class组件

本质是ES6 class 语法

举例：
```
    class A extends React.Component{
        constructor(props){
            super(props)
            this.state = {
                firstName: 'zhang',
                lastName: 'san',
            }
        }
        get name(){return `${this.state.firstName} ${this.state.lastName}`}
        set name(newName){
            const [firstName,lastName] = newName.split(' ')
            this.setState({firstName,lastName})
        }
        render(){
            return <h1>{this.name}</h1>
        }
    }
```
state & props 均可如上作计算属性

