# 7.3 Filter Users

Tenemos un formulario con dos filtros: nombre y categoría
Y un listado de usuarios.

`app/user/user.component.ts`
```ts
export class UserComponent implements OnInit {

  users$: Observable<User[]>;
  categoryFilter: any;
  nameFilter: any;

  constructor(private userService: UserService) { }

  ngOnInit() {
    this.users$ = this.userService.getUsers();
    // this.users$ = this.solution();
   }

  onCategoryFilter(category: string) {}

  onNameFilter(name: string) { }

  private solution(): Observable<User[]> {
  }
}
```
<br>

## 7.3.1 Ejercicio
---

Implementar `solution` para que devuelva un observable con la lista de usuarios filtrada.

Cada vez que cambiamos un filtro, se actualiza el listado de usuarios.

<br>

## 7.3.2 Solución
---

`app/user/user.component.ts`
```ts
  private solution(): Observable<User[]> {
    return combineLatest([
      this.userService.getUsers(),
      this.nameFilterSubject$.pipe(
        debounceTime(300)
      ),
      this.categoryFilterSubject$
    ]).pipe(
      map(
        ([users, name, category]) => {
          console.log(users, name, category);
          let filteredUsers = users.filter(
            user => user.name.toLowerCase().includes(name.toLowerCase())
          );
          if (category) {
            filteredUsers = filteredUsers.filter(user => user.category === category);
          }
          return filteredUsers;
        }
      )
    );
  }
```
<br>