# Principios SOLID en Java

Los principios SOLID son un conjunto de cinco directrices de diseño orientado a objetos que, cuando se aplican correctamente, resultan en sistemas más robustos y fáciles de mantener. A continuación se presenta cada principio con ejemplos prácticos en Java.

---

## S – Single Responsibility Principle (SRP)

**Definición:**  
Una clase debe ocuparse de **una sola cosa**. Si una clase tiene más de una razón para ser modificada, está asumiendo demasiadas responsabilidades.

**Ejemplo (diseño problemático):**

```java
class Factura {
    private double total;

    public double calcularTotal(double precio, int cantidad) {
        return precio * cantidad;
    }

    // Responsabilidad extra: envío por correo → viola SRP
    public void enviarPorEmail(String destinatario) {
        // lógica para enviar el email con la factura
        System.out.println("Enviando factura a: " + destinatario);
    }
}
```

Aquí `Factura` calcula valores de negocio **y** gestiona el envío de emails. Dos razones para cambiar.

**Ejemplo (aplicando SRP):**

```java
class Factura {
    private double total;

    public double calcularTotal(double precio, int cantidad) {
        return precio * cantidad;
    }
}

class FacturaEmailSender {
    public void enviar(Factura factura, String destinatario) {
        // lógica de envío desacoplada del cálculo
        System.out.println("Enviando factura a: " + destinatario);
    }
}
```

Ahora cada clase tiene una única razón de cambio: `Factura` si cambia la lógica de negocio, y `FacturaEmailSender` si cambia el mecanismo de envío.

---

## O – Open/Closed Principle (OCP)

**Definición:**  
El código existente no debería modificarse para agregar nueva funcionalidad. Las clases deben estar **abiertas para ser extendidas** y **cerradas frente a modificaciones**.

**Ejemplo (violando OCP):**

```java
class GeneradorReporte {
    public void exportar(String datos, String formato) {
        if (formato.equals("PDF")) {
            System.out.println("Exportando como PDF...");
        } else if (formato.equals("CSV")) {
            System.out.println("Exportando como CSV...");
        }
        // cada nuevo formato obliga a tocar este método
    }
}
```

Agregar un formato `EXCEL` implica editar una clase que ya funcionaba correctamente.

**Ejemplo (aplicando OCP):**

```java
interface ExportadorReporte {
    void exportar(String datos);
}

class ExportadorPdf implements ExportadorReporte {
    public void exportar(String datos) {
        System.out.println("Generando PDF con: " + datos);
    }
}

class ExportadorCsv implements ExportadorReporte {
    public void exportar(String datos) {
        System.out.println("Generando CSV con: " + datos);
    }
}

class ExportadorExcel implements ExportadorReporte {
    public void exportar(String datos) {
        System.out.println("Generando Excel con: " + datos);
    }
}

class GeneradorReporte {
    public void exportar(String datos, ExportadorReporte exportador) {
        exportador.exportar(datos);
    }
}
```

Se puede agregar `ExportadorExcel` o cualquier otro formato sin tocar `GeneradorReporte`.

---

## L – Liskov Substitution Principle (LSP)

**Definición:**  
Un objeto de una subclase debe poder usarse en cualquier lugar donde se use su superclase, **sin alterar el comportamiento esperado** del programa.

**Ejemplo (violando LSP):**

```java
class Ave {
    public void volar() {
        System.out.println("Volando...");
    }
}

class Pinguino extends Ave {
    @Override
    public void volar() {
        // El pingüino no vuela → rompe la expectativa del método
        throw new UnsupportedOperationException("Los pingüinos no vuelan");
    }
}
```

Si un método recibe un `Ave` y llama a `volar()`, explotará al recibir un `Pinguino`. La herencia está mal modelada.

**Ejemplo (aplicando LSP):**

```java
interface Ave {
    void moverse();
}

interface AveVoladora extends Ave {
    void volar();
}

class Aguila implements AveVoladora {
    public void moverse() {
        System.out.println("El águila se desplaza.");
    }

    public void volar() {
        System.out.println("El águila vuela alto.");
    }
}

class Pinguino implements Ave {
    public void moverse() {
        System.out.println("El pingüino nada y camina.");
    }
}
```

Cada clase cumple exactamente con el contrato de la interfaz que implementa, sin sorpresas.

---

## I – Interface Segregation Principle (ISP)

**Definición:**  
Ninguna clase debería verse obligada a depender de métodos que no utiliza. Es preferible tener **varias interfaces pequeñas y específicas** que una sola interfaz grande.

**Ejemplo (violando ISP):**

```java
interface DispositivoMultifuncion {
    void imprimir(String documento);
    void escanear(String documento);
    void enviarFax(String numero);
}

class Impreso raBasica implements DispositivoMultifuncion {
    public void imprimir(String documento) {
        System.out.println("Imprimiendo: " + documento);
    }

    public void escanear(String documento) {
        throw new UnsupportedOperationException("Esta impresora no escanea");
    }

    public void enviarFax(String numero) {
        throw new UnsupportedOperationException("Esta impresora no envía fax");
    }
}
```

La impresora básica se ve forzada a "implementar" funciones que no tiene.

**Ejemplo (aplicando ISP):**

```java
interface Imprimible {
    void imprimir(String documento);
}

interface Escaneable {
    void escanear(String documento);
}

interface EnviaFax {
    void enviarFax(String numero);
}

class ImpresoraBasica implements Imprimible {
    public void imprimir(String documento) {
        System.out.println("Imprimiendo: " + documento);
    }
}

class ImpresoraOficina implements Imprimible, Escaneable, EnviaFax {
    public void imprimir(String documento) { System.out.println("Imprimiendo."); }
    public void escanear(String documento) { System.out.println("Escaneando."); }
    public void enviarFax(String numero)   { System.out.println("Enviando fax."); }
}
```

`ImpresoraBasica` solo implementa lo que realmente puede hacer.

---

## D – Dependency Inversion Principle (DIP)

**Definición:**  
Los módulos de alto nivel no deben conocer los detalles internos de los módulos de bajo nivel. Ambos deben depender de **abstracciones**, no de implementaciones concretas.

**Ejemplo (sin DIP):**

```java
class NotificadorEmail {
    public void enviar(String mensaje) {
        System.out.println("Email: " + mensaje);
    }
}

class SistemaAlertas {
    private NotificadorEmail notificador = new NotificadorEmail(); // acoplamiento rígido

    public void lanzarAlerta(String mensaje) {
        notificador.enviar(mensaje);
    }
}
```

Si mañana se necesita notificar por SMS, hay que modificar `SistemaAlertas` directamente.

**Ejemplo (aplicando DIP):**

```java
interface Notificador {
    void enviar(String mensaje);
}

class NotificadorEmail implements Notificador {
    public void enviar(String mensaje) {
        System.out.println("Email: " + mensaje);
    }
}

class NotificadorSms implements Notificador {
    public void enviar(String mensaje) {
        System.out.println("SMS: " + mensaje);
    }
}

class SistemaAlertas {
    private Notificador notificador;

    public SistemaAlertas(Notificador notificador) { // inyección de dependencia
        this.notificador = notificador;
    }

    public void lanzarAlerta(String mensaje) {
        notificador.enviar(mensaje);
    }
}
```

`SistemaAlertas` no sabe si el mensaje va por email o SMS. Solo conoce la abstracción `Notificador`, lo que permite cambiar el canal sin tocar la lógica de alertas.

---

## Conclusión

- Aplicar SOLID de forma consistente convierte el código en algo **predecible y confiable**: cada pieza tiene un propósito claro y los cambios en una parte no provocan efectos inesperados en otras.
- Estos principios no son recetas rígidas; son **herramientas de razonamiento** que ayudan a detectar señales de alerta en el diseño, como clases demasiado grandes, jerarquías de herencia forzadas o dependencias difíciles de sustituir.
- Un código que respeta SOLID es naturalmente más fácil de **probar con pruebas unitarias**, ya que las responsabilidades están bien delimitadas y las dependencias se pueden reemplazar por objetos de prueba (mocks).
- A largo plazo, invertir tiempo en estos principios reduce significativamente el costo de mantenimiento y permite que los equipos incorporen nuevas funcionalidades con **menos miedo a romper lo que ya funciona**.
