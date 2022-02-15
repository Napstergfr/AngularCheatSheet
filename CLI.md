# Angular CLI CheatSheet

### * To Install Angular CLI
```Bash
npm install -g @angular/cli
```
> _NOTE: use -g flag for global installation & don't use -g flag to install locally_

### * To Create New Angular App
```Bash
ng new napster-app
```
> _NOTE: ng new napster-app(Your App Name)_

### * To Generate New Routing Module
```Bash
ng g m auth/AuthRouting --route --flat --module=auth
```
> _NOTE: ng g m auth(Folder Name)/AuthRouting(New routing module name) --route --flat(To ignore creation extra folder for routing module) --module=auth (Module name to attach with) Please user the code below if you want use this route as a child route_
```Javascript
const routes: Routes = []

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
```
### * To Generate New Module
```Bash
ng g m AuthModule
or
ng g m AuthModule --flat
or
ng g m AuthModule --module=App
```
> _NOTE: ng g m AuthModule(New Module Name) "--flat" this flag is to put the module into the directory root. The flag --module will automatically import this module to app module_

### * To Generate New Interceptor
```Bash
 ng g interceptor shared/interceptors/NewInterceptor
```
> _NOTE:  ng g interceptor shared/interceptors/NewInterceptor(New interceptor name)_

### * To Generate New Component
```Bash
ng g c user/AddUser
```
> _NOTE: ng g c user/AddUser(New component name)_

### * To Generate New Guard
```Bash
ng g guard shared/Authguard
```
> _NOTE: ng g guard shared/Authguard(New Guard Name)_

### * To Generate New Service
```Bash
ng g s shared/service/Auth
```
> _NOTE: ng g s shared/service/Auth(New service name)_

### * To Install flexLayout/gdColumn & cdk
```Bash
npm i -s @angular/flex-layout @angular/cdk
```
> _NOTE: cdk is needed for flexLayout_

> Don't Forget to import in app.module.ts
```Javascript
import { FlexLayoutModule } from '@angular/flex-layout';
...

@NgModule({
    ...
    imports: [ FlexLayoutModule ],
    ...
});
```
