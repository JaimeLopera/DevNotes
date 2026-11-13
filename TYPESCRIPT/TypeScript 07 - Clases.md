# 07 - Clases

> [!info] ¿Qué es?
> TypeScript añade a las **clases** de JavaScript tipos en propiedades y métodos, **modificadores de acceso** (`public`, `private`, `protected`), propiedades `readonly`, clases **abstractas** y la posibilidad de **implementar interfaces**. Así el diseño orientado a objetos es más seguro y explícito.

---

## 1. Antes de empezar

- Conviene conocer las clases de JavaScript (`class`, `constructor`, `extends`): [[JavaScript 18 - JavaScript moderno|JavaScript moderno]].
- Y haber leído [[TypeScript 03 - Interfaces y type]].
- La teoría de la programación orientada a objetos está en [[PHP 14 - POO|POO en PHP]] y [[Python 14 - POO|POO en Python]].

---

## 2. Concepto fundamental

Una clase es una **plantilla** para crear objetos con datos (propiedades) y comportamiento (métodos). TypeScript obliga a **declarar** las propiedades con su tipo y controla **quién puede acceder** a ellas.

```typescript
class CuentaBancaria {
  private saldo: number = 0;           // solo accesible desde dentro

  constructor(public titular: string) {} // crea la propiedad titular

  ingresar(cantidad: number): void {
    if (cantidad <= 0) throw new Error("Cantidad no válida");
    this.saldo += cantidad;
  }

  verSaldo(): number {
    return this.saldo;
  }
}

const cuenta = new CuentaBancaria("Ana");
cuenta.ingresar(100);
cuenta.saldo = 1_000_000; // ✘ Error: 'saldo' es private
```

---

## 3. Sintaxis / estructura

```typescript
class NombreClase {
  propiedad: tipo;

  constructor(parametro: tipo) {
    this.propiedad = parametro;
  }

  metodo(param: tipo): tipoRetorno {
    // ...
  }
}
```

---

## 4. Elementos / propiedades / características

### 4.1 Modificadores de acceso

| Modificador | Accesible desde |
|---|---|
| `public` (por defecto) | Cualquier sitio |
| `protected` | La clase y sus subclases |
| `private` | Solo la propia clase |
| `#propiedad` | Solo la propia clase (privado **real** de JavaScript, también al ejecutar) |

### 4.2 `readonly`

```typescript
class Producto {
  readonly id: number;
  constructor(id: number) {
    this.id = id;   // ✔ se puede asignar en el constructor
  }
}
```

### 4.3 Propiedades en el constructor (atajo)

```typescript
// Forma larga
class Usuario {
  nombre: string;
  private email: string;
  constructor(nombre: string, email: string) {
    this.nombre = nombre;
    this.email = email;
  }
}

// Atajo: el modificador en el parámetro crea y asigna la propiedad
class Usuario2 {
  constructor(public nombre: string, private email: string) {}
}
```

### 4.4 Herencia

```typescript
class Animal {
  constructor(protected nombre: string) {}
  presentarse(): string {
    return `Soy ${this.nombre}`;
  }
}

class Perro extends Animal {
  override presentarse(): string {
    return `${super.presentarse()} y digo guau`;
  }
}
```

`override` deja claro que reescribes un método del padre (y avisa si el padre no lo tiene).

### 4.5 Implementar interfaces

```typescript
interface Almacen {
  guardar(clave: string, valor: string): void;
  leer(clave: string): string | null;
}

class AlmacenLocal implements Almacen {
  guardar(clave: string, valor: string) {
    localStorage.setItem(clave, valor);
  }
  leer(clave: string) {
    return localStorage.getItem(clave);
  }
}
```

### 4.6 Clases abstractas

No se pueden instanciar; sirven de base y obligan a las hijas a implementar ciertos métodos:

```typescript
abstract class Figura {
  abstract area(): number;
  describir() {
    return `Área: ${this.area().toFixed(2)}`;
  }
}

class Circulo extends Figura {
  constructor(private radio: number) { super(); }
  area() { return Math.PI * this.radio ** 2; }
}
```

### 4.7 Miembros estáticos y getters

```typescript
class Temperatura {
  static readonly CERO_ABSOLUTO = -273.15;
  constructor(private celsius: number) {}
  get fahrenheit(): number {
    return this.celsius * 1.8 + 32;
  }
}

Temperatura.CERO_ABSOLUTO;
new Temperatura(20).fahrenheit; // 68
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```typescript
class Contador {
  private valor = 0;
  incrementar() { this.valor++; }
  get actual() { return this.valor; }
}

const c = new Contador();
c.incrementar();
console.log(c.actual); // 1
```

### 5.2 Ejemplo habitual: servicio de API con interfaz

```typescript
interface Tarea {
  id: number;
  titulo: string;
  hecha: boolean;
}

class ServicioTareas {
  constructor(private readonly urlBase: string) {}

  async listar(): Promise<Tarea[]> {
    const res = await fetch(`${this.urlBase}/tareas`);
    return res.json();
  }

  async crear(titulo: string): Promise<Tarea> {
    const res = await fetch(`${this.urlBase}/tareas`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ titulo }),
    });
    return res.json();
  }
}

const servicio = new ServicioTareas("/api");
const tareas = await servicio.listar();
```

---

## 6. Buenas prácticas

- **`private` por defecto**; expón solo lo necesario.
- **`readonly`** para lo que no cambia tras crear el objeto.
- **Usa el atajo del constructor** para reducir código repetido.
- **Programa contra interfaces** (`implements`): facilita cambiar implementaciones y testear.
- **No abuses de las clases**: en TypeScript, muchas veces bastan funciones y objetos simples.
- **Composición antes que herencia** profunda.

---

## 7. Diferencias importantes

### 7.1 `private` vs. `#privado`

| | `private` | `#privado` |
|---|---|---|
| Lo comprueba | Solo TypeScript (al compilar) | JavaScript (también al ejecutar) |
| Accesible con trucos en ejecución | Sí | No |

### 7.2 `interface` vs. clase abstracta

| | `interface` | Clase abstracta |
|---|---|---|
| Puede tener código | No | Sí |
| Una clase puede tener varias | Sí (`implements A, B`) | No (solo un `extends`) |
| Existe al ejecutar | No | Sí |

### 7.3 `protected` vs. `private`

`protected` es visible en las subclases; `private`, no.

---

## 8. Casos especiales

### 8.1 *Property has no initializer*

Con `strict`, las propiedades deben inicializarse en la declaración o en el constructor. Si se asignan más tarde de forma segura, usa `!`: `nombre!: string` (con cuidado).

### 8.2 Perder `this` en callbacks

```typescript
boton.addEventListener("click", objeto.metodo);   // this se pierde
boton.addEventListener("click", () => objeto.metodo()); // ✔
```

O define el método como función flecha: `metodo = () => { ... }`.

---

## 9. Resumen

- Las propiedades se declaran con tipo; `public`, `protected`, `private` controlan el acceso.
- `readonly` impide cambios tras el constructor.
- Atajo: `constructor(private x: number) {}` crea la propiedad.
- `implements` obliga a cumplir una interfaz; `abstract` define una base no instanciable.
- `override` al reescribir métodos heredados.
- Siguiente paso: [[TypeScript 08 - Utility types]].
