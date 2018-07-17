## MobX(3)--Change Observables

改变状态，以及异步操作。

#### action

用法：
- `action(fn)`
- `action(name, fn)`
- `@action classMethod() {}`
- `@action(name) classMethod() {}`
- `@action boundClassMethod = (args) => { body }`
- `@action(name) boundClassMethod = (args) => { body }`
- `@action.bound classMethod() {}`

```js
class Ticker {
    @observable tick = 0

    @action.bound
    increment() {
        this.tick++
    }
}
```

#### action配合Promise

```js
// 一定要通过action来修改状态
mobx.configure({ enforceActions: true })

class Store {
    @observable githubProjects = []
    @observable state = 'pending'

    @action
    fetchProjects() {
        this.githubProjects = []
        this.state = 'pending'
        fetchGithubProjectsSomehow()
            .then(this.fetchProjectsSuccess, this.fetchProjectsError)
    }

    @action.bound
    fetchProjectsSuccess(projects) {
        const filteredProjects = somePreprocessing(projects)
        this.githubProjects = filteredProjects
        this.state = 'done'
    }

    @action.bound
    fetchProjectsError(error) {
        this.state = 'error'
    }
}
```

#### action 配合 async/await

```js
// 一定要通过action来修改状态
mobx.configure({ enforceActions: true })

class Store {
    @observable githubProjects = []
    @observable state = 'pending'

    @action
    async fetchProjects() {
        this.githubProjects = []
        this.state = 'pending'
        try {
            const projects = await fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            runInAction(() => {
                this.state = 'done'
                this.githubProjects = filteredProjects
            })
        } catch (error) {
            runInAction(() => {
                this.state = 'error'
            })
        }
    }
}
```
