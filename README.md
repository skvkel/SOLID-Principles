# Principios SOLID con ejemplos
## Single Responsability (responsabilidad única) SRP
Este principio tiene un nombre algo confuso. Una mejor definición del principio
sería:

**Un módulo solo debe ser responsable de un solo actor**

La definición de una clase solo debería cambiar por un solo motivo nos llega
a confusiones, ya que es muy abstracto decir que un módulo tenga una única 
responsabilidad.

Sin embargo, con la nueva definición se aclara un poco la finalidad del principio.
Nos dice que un módulo/clase solo debe ser responsable de un actor, y no de 
varios.
Imaginemos el ejemplo de una clase Empleado, que tiene sus atributos privados
y tiene unos métodos de:
- calcular_pago
- reportar_horas
- guardar_empleado

Estos tres métodos se corresponden a tres actores distintos (el primero al 
departamento de finanzas, el segundo al departamendo de RRHH y el 
tercero al repositorio que se encarga de almacenar un empleado en base de datos)
Esto quiere decir que si necesitamos cambiar el módulo/clase Empleado por 
alguna de esos métodos, estará cambiando por más de un actor. Un cambio en el 
método calcular_pago podría impactar sobre el actor que utiliza el método
reportar_horas. Es por ello que el principio se define mejor con que cada 
módulo/clase debe ser responsable de un solo actor.

En este caso, al ser Empleado una entidad, debería tener únicamente los atributos
privados del empleado y los métodos públicos para acceder a sus atributos.

Esta implementación violaría el principio SRP
```python
class Empleado:
    def __init__(self, nombre, horas_trabajadas, tarifa):
        self.nombre = nombre
        self.horas_trabajadas = horas_trabajadas
        self.tarifa = tarifa

    def calcular_pago(self):
        return self.horas_trabajadas * self.tarifa

    def reportar_horas(self):
        print(f"{self.nombre} ha trabajado {self.horas_trabajadas} horas.")

    def guardar(self):
        with open('empleado.txt', 'w') as file:
            file.write(f"{self.nombre}, {self.horas_trabajadas}, {self.tarifa}")
```

Esta otra implementación, a su vez, cumpliría el principio SRP.

```python
@dataclass(frozen=True)
class Empleado:
    nombre: str
    horas_trabajadas: int
    tarifa: float

class CalculadoraPago:
    @staticmethod
    def calcular_pago(empleado):
        return empleado.horas_trabajadas * empleado.tarifa

class ReportadorHoras:
    @staticmethod
    def reportar_horas(empleado):
        print(f"{empleado.nombre} ha trabajado {empleado.horas_trabajadas} horas.")

class GuardarEmpleado:
    @staticmethod
    def guardar_empleado(empleado):
        with open('empleado.txt', 'w') as file:
            file.write(f"{empleado.nombre}, {empleado.horas_trabajadas}, {empleado.tarifa}")

            
empleado = Empleado("Juan", 40, 15)
pago = CalculadoraPago.calcular_pago(empleado)
print(f"El pago es: {pago}")

ReportadorHoras.reportar_horas(empleado)
GuardarEmpleado.guardar(empleado)

```

## Open/Close (Abierto/Cerrado) OCP
Este principio nos dice que una clase está abierta para extender pero cerrada
para modificar. En otras palabras: una clase implementada no se puede modificar,
pero si se debe poder extender (**heredar**).

Un ejemplo de violación de este principio:

```python
class Calculadora:
    def calcular(self, a, b, operacion):
        if operacion == "suma":
            return a + b
        elif operacion == "resta":
            return a - b
        elif operacion == "multiplicacion":
            return a * b
        elif operacion == "division":
            return a / b
        else:
            raise ValueError("Operación no soportada")

calculadora = Calculadora()
resultado = calculadora.calcular(10, 5, "suma")
```
Si ahora necesitásemos una potencia, nuestra clase no soportaría esta operación
y deberíamos modificar la clase (que puede estar ya testada y funcionando 
correctamente).

La solución para esto sería la siguiente:
```python
# Definimos una interfaz para la operación
class Operacion(ABC):
    @abstractmethod
    def ejecutar(self, a, b):
        pass

# Implementamos operaciones concretas
class Suma(Operacion):
    def ejecutar(self, a, b):
        return a + b

class Resta(Operacion):
    def ejecutar(self, a, b):
        return a - b

class Multiplicacion(Operacion):
    def ejecutar(self, a, b):
        return a * b

class Division(Operacion):
    def ejecutar(self, a, b):
        return a / b

# La clase Calculadora usa la composición para trabajar con operaciones
class Calculadora:
    def __init__(self, operacion: Operacion):
        self.operacion = operacion

    def calcular(self, a, b):
        return self.operacion.ejecutar(a, b)

# Le inyectamos una concreción a la clase Calculadora
calculadora = Calculadora(Suma())
resultado = calculadora.calcular(10, 5)
print(resultado)  # Output: 15

# Para agregar una nueva operación, simplemente creamos una nueva clase sin 
# modificar las existentes
class Potencia(Operacion):
    def ejecutar(self, a, b):
        return a ** b

# Ahora podemos usar la nueva operación sin modificar la clase Calculadora
calculadora = Calculadora(Potencia())
resultado = calculadora.calcular(2, 3)
print(resultado)  # Output: 8
```
Para cumplir este principio, hemos utilizado el otro principio SOLID "inversión
de dependencias", con el que ahora la clase Calculadora no depende directamente
de una concreción, sino que depende de una abstracción.
## Sustitución de Liskov LSP
Este principio nos dice que si B es un subtipo de A, entonces los objetos de 
tipo A pueden ser sustituidos por objetos de tipo B sin perjuicios. Lo que es
lo mismo, una clase derivada puede sustituir a la clase base.

El siguiente ejemplo viola el principio:
````python
class Pajaro:
    def volar(self):
        print("Volando")

class Pinguino(Pajaro):
    def volar(self):
        raise Exception("Pinguinos no pueden volar")

````
Esto es debido porque los pinguinos no pueden volar, y al hacer un override del
método volar, el sistema deja de funcionar correctamente.

Una corrección podría ser extraer el método volar a los pájaros que pueden volar:

````python
# Todos los pájaros se pueden mover andando
class Pajaro:
    def andar(self):
        print("Andando")

# Estos pájaros pueden volar
class PajaroVolador(Pajaro):
    def volar(self):
        print("Volando")

# El pinguino es un pájaro que solo puede andar
class Pinguino(Pajaro):
    def andar(self):
        print("Andando con dos patas")

class Loro(PajaroVolador):
    def volar(self):
        print("Volando")

# Ejemplo de uso:
def hacer_volar_pajaro(pajaro: PajaroVolador):
    pajaro.volar()

# Esto ahora es seguro:
loro = Loro()
hacer_volar_pajaro(loro)

# Esto también es seguro:
pinguino = Pinguino()
Pinguino.andar()  # No intenta volar, sólo nada. No tiene método volar

````

## Inversión de dependencias
Este principio nos dice que no debemos de depender de concreciones o implementaciones
sino de abstracciones. El ejemplo del principio anterior ilustra muy bien este
principio, ya que la clase Calculadora tiene una inyección de una clase
abstracta, cuyas implementaciones deben implementar el método "ejecutar".
## Segregación de interfaces
Este principio se refiere a que las clases deben de tener justo lo necesario y 
no más. El incluir numerosos métodos en una clase que puede servir para muchas
cosas, es un error. Deberíamos segregar las interfaces en más de una interfaz.

El siguiente ejemplo viola el principio de Segregación de interfaces:

```python
class Operaciones:
    def suma(self, a, b):
        pass

    def resta(self, a, b):
        pass

    def multiplicacion(self, a, b):
        pass

    def division(self, a, b):
        pass

    def potencia(self, a, b):
        pass

class CalculadoraSimpleImp(Operaciones):
    def suma(self, a, b):
        return a + b

    def resta(self, a, b):
        return a - b

    def multiplicacion(self, a, b):
        return a * b

    def division(self, a, b):
        raise NotImplementedError("Esta operación no está soportada")

    def potencia(self, a, b):
        raise NotImplementedError("Esta operación no está soportada")

calculadora = CalculadoraSimpleImp()
print(calculadora.suma(10, 5))  # Output: 15

```
La clase Operaciones tiene un montón de métodos que pueden ser separados en 
otras interfaces.

El siguiente ejemplo es una corrección del ejemplo anterior para cumplir con 
el principio de Segregación de interfaces:
```python
# Interfaces más pequeñas y específicas
class SumaOperacion(ABC):
    @abstractmethod
    def suma(self, a, b):
        pass

class RestaOperacion(ABC):
    @abstractmethod
    def resta(self, a, b):
        pass

class MultiplicacionOperacion(ABC):
    @abstractmethod
    def multiplicacion(self, a, b):
        pass

class DivisionOperacion(ABC):
    @abstractmethod
    def division(self, a, b):
        pass

class PotenciaOperacion(ABC):
    @abstractmethod
    def potencia(self, a, b):
        pass

# Implementa las distintas operaciones
class CalculadoraSimpleImp(SumaOperacion, RestaOperacion, MultiplicacionOperacion):
    def suma(self, a, b):
        return a + b

    def resta(self, a, b):
        return a - b

    def multiplicacion(self, a, b):
        return a * b

# Clases que implementan otras interfaces
class CalculadoraAvanzadaImp(SumaOperacion, RestaOperacion, 
                          MultiplicacionOperacion, DivisionOperacion, 
                          PotenciaOperacion):
    def suma(self, a, b):
        return a + b

    def resta(self, a, b):
        return a - b

    def multiplicacion(self, a, b):
        return a * b

    def division(self, a, b):
        return a / b

    def potencia(self, a, b):
        return a ** b

calculadora_simple = CalculadoraSimpleImp()
print(calculadora_simple.suma(10, 5))  # Output: 15

calculadora_avanzada = CalculadoraAvanzadaImp()
print(calculadora_avanzada.potencia(2, 3))  # Output: 8

```
