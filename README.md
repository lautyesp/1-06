Problema 1: El Sistema de Envíos Todopoderoso
Análisis del Problema
La clase OrderService incumple el principio Single Responsibility Principle (SRP) porque se encarga de varias tareas al mismo tiempo:
Calcular costos de envío.
Procesar pagos.
Enviar notificaciones.
Coordinar el procesamiento del pedido.
También incumple el principio Open/Closed Principle (OCP) porque para agregar un nuevo método de envío o pago es necesario modificar la clase agregando nuevos bloques if o else if.

Soluciones Propuestas
Aplicación de SRP
Se separaron las responsabilidades en clases independientes:
ShippingMethod: cálculo del costo de envío.
PaymentMethod: procesamiento de pagos.
NotificationService: envío de notificaciones.
OrderService: coordinación del flujo del pedido.
Aplicación de OCP
Se implementaron interfaces para permitir extender el sistema sin modificar código existente:
interface ShippingMethod {
    calculateCost(): number;
}

interface PaymentMethod {
    pay(amount: number): void;
}

Luego se crearon distintas implementaciones para envíos y pagos.

Código Refactorizado
class Order {
    constructor(
        public id: string,
        public totalAmount: number
    ) {}
}

/* =========================
   ENVÍOS
========================= */

interface ShippingMethod {
    calculateCost(): number;
}

class StandardShipping implements ShippingMethod {
    calculateCost(): number {
        return 10;
    }
}

class ExpressShipping implements ShippingMethod {
    calculateCost(): number {
        return 25;
    }
}

class DroneShipping implements ShippingMethod {
    calculateCost(): number {
        return 50;
    }
}

/* =========================
   PAGOS
========================= */

interface PaymentMethod {
    pay(amount: number): void;
}

class PaypalPayment implements PaymentMethod {
    pay(amount: number): void {
        console.log(`Procesando pago de $${amount} vía PayPal`);
    }
}

class CreditCardPayment implements PaymentMethod {
    pay(amount: number): void {
        console.log(`Cargando $${amount} a la tarjeta de crédito`);
    }
}

/* =========================
   NOTIFICACIONES
========================= */

class NotificationService {
    send(orderId: string): void {
        console.log(
            `Email enviado: Su pedido ${orderId} ha sido procesado`
        );
    }
}

/* =========================
   ORDER SERVICE
========================= */

class OrderService {

    constructor(
        private paymentMethod: PaymentMethod,
        private shippingMethod: ShippingMethod,
        private notificationService: NotificationService
    ) {}

    processOrder(order: Order): void {

        const shippingCost =
            this.shippingMethod.calculateCost();

        const total =
            order.totalAmount + shippingCost;

        this.paymentMethod.pay(total);

        this.notificationService.send(order.id);
    }
}

/* =========================
   EJEMPLO DE USO
========================= */

const order = new Order("001", 100);

const service = new OrderService(
    new PaypalPayment(),
    new ExpressShipping(),
    new NotificationService()
);

service.processOrder(order);


Pruebas Realizadas
Se probaron los siguientes escenarios:
Pedido con envío estándar.
Pedido con envío express.
Pago mediante PayPal.
Pago mediante tarjeta de crédito.
Envío correcto de notificaciones.
Cálculo correcto del total del pedido.
Todas las pruebas fueron exitosas.

Conclusión
La solución cumple con los principios SRP y OCP al separar responsabilidades y permitir extender el sistema mediante nuevas implementaciones de interfaces sin modificar las clases existentes.
Beneficios obtenidos:
Mayor mantenibilidad.
Menor acoplamiento.
Código más escalable.
Facilidad para agregar nuevas formas de envío y pago.
Mejor capacidad para realizar pruebas automatizadas.

=============================================================================================================================================================================================================================

Problema 2: El Procesador de Documentos Rebelde
Análisis del Problema
El sistema incumple el principio Interface Segregation Principle (ISP) porque obliga a todos los documentos a implementar métodos que quizás no necesitan.
La interfaz original:
interface DocumentHandler {
    open(): void;
    edit(): void;
    save(): void;
}

obliga a que un PDF protegido implemente edit() y save(), aunque dichas operaciones no sean válidas para ese tipo de archivo.
También incumple el principio Liskov Substitution Principle (LSP) porque una instancia de PDFDocument no puede reemplazar correctamente a un DocumentHandler. Cuando el cliente llama a edit() o save(), el programa genera excepciones y falla.

Soluciones Propuestas
Aplicación de ISP
Se divide la interfaz en interfaces más pequeñas y específicas:
Openable: documentos que pueden abrirse.
Editable: documentos que pueden editarse.
Savable: documentos que pueden guardarse.
De esta forma cada clase implementa únicamente las operaciones que realmente soporta.
Aplicación de LSP
Los clientes trabajarán únicamente con las capacidades que necesitan.
Un documento PDF protegido implementará solamente Openable, mientras que un documento Word implementará las tres interfaces.
Así ninguna implementación deberá lanzar errores por operaciones que no puede realizar.

Código Refactorizado
/* =========================
   INTERFACES
========================= */

interface Openable {
    open(): void;
}

interface Editable {
    edit(): void;
}

interface Savable {
    save(): void;
}

/* =========================
   DOCUMENTO WORD
========================= */

class WordDocument
    implements Openable, Editable, Savable {

    open(): void {
        console.log("Abriendo documento Word...");
    }

    edit(): void {
        console.log("Editando texto...");
    }

    save(): void {
        console.log("Guardando cambios...");
    }
}

/* =========================
   PDF PROTEGIDO
========================= */

class PDFDocument implements Openable {

    open(): void {
        console.log("Abriendo PDF protegido...");
    }
}

/* =========================
   PROCESADORES
========================= */

function procesarDocumentoEditable(
    doc: Openable & Editable & Savable
): void {

    doc.open();
    doc.edit();
    doc.save();
}

function visualizarDocumento(
    doc: Openable
): void {

    doc.open();
}

/* =========================
   EJEMPLO DE USO
========================= */

const word = new WordDocument();
procesarDocumentoEditable(word);

const pdf = new PDFDocument();
visualizarDocumento(pdf);


Pruebas Realizadas
Se verificaron los siguientes escenarios:
Apertura de documentos Word.
Edición de documentos Word.
Guardado de documentos Word.
Apertura de PDFs protegidos.
Ejecución sin errores ni excepciones.
Sustitución segura de implementaciones según sus capacidades.
Todos los casos funcionaron correctamente.

Conclusión
La solución cumple con los principios ISP y LSP.
Beneficios obtenidos:
Las clases implementan únicamente los métodos que necesitan.
Se eliminan excepciones innecesarias.
Las implementaciones pueden sustituirse sin romper el sistema.
El código es más mantenible y extensible.
Se mejora la claridad de las responsabilidades de cada tipo de documento.


=============================================================================================================================================================================================================================

Problema 3: El Interruptor Rígido
Análisis del Problema
El sistema incumple el principio Dependency Inversion Principle (DIP) porque la clase Switch depende directamente de una implementación concreta (TraditionalBulb).
this.bulb = new TraditionalBulb();

Esto genera un fuerte acoplamiento entre ambas clases.
Si en el futuro se quisiera utilizar otro dispositivo, como una lámpara inteligente o un ventilador, sería necesario modificar el código de la clase Switch, lo que dificulta la escalabilidad y el mantenimiento del sistema.

Soluciones Propuestas
Aplicación de DIP
Se crea una abstracción llamada SwitchableDevice que define el comportamiento común de cualquier dispositivo que pueda encenderse o apagarse.
interface SwitchableDevice {
    turnOn(): void;
    turnOff(): void;
}

La clase Switch dependerá de esta interfaz en lugar de depender de una implementación concreta.
Inyección de Dependencias
El dispositivo será recibido desde el exterior mediante el constructor.
De esta manera:
Switch no necesita saber qué dispositivo controla.
Es posible utilizar bombillas, luces inteligentes, ventiladores u otros dispositivos sin modificar la clase.
Se reduce el acoplamiento entre módulos.

Código Refactorizado
/* =========================
   ABSTRACCIÓN
========================= */

interface SwitchableDevice {
    turnOn(): void;
    turnOff(): void;
}

/* =========================
   DISPOSITIVOS
========================= */

class TraditionalBulb implements SwitchableDevice {

    turnOn(): void {
        console.log(
            "Bombilla tradicional encendida... consumiendo mucha energía."
        );
    }

    turnOff(): void {
        console.log("Bombilla tradicional apagada.");
    }
}

class SmartLight implements SwitchableDevice {

    turnOn(): void {
        console.log("Luz inteligente encendida.");
    }

    turnOff(): void {
        console.log("Luz inteligente apagada.");
    }
}

class Fan implements SwitchableDevice {

    turnOn(): void {
        console.log("Ventilador encendido.");
    }

    turnOff(): void {
        console.log("Ventilador apagado.");
    }
}

/* =========================
   SWITCH
========================= */

class Switch {

    constructor(
        private device: SwitchableDevice
    ) {}

    operate(action: string): void {

        if (action === "on") {
            this.device.turnOn();
        } else {
            this.device.turnOff();
        }
    }
}

/* =========================
   EJEMPLOS DE USO
========================= */

const bulbSwitch = new Switch(
    new TraditionalBulb()
);

bulbSwitch.operate("on");

const smartLightSwitch = new Switch(
    new SmartLight()
);

smartLightSwitch.operate("on");

const fanSwitch = new Switch(
    new Fan()
);

fanSwitch.operate("off");


Pruebas Realizadas
Se verificaron los siguientes escenarios:
Encendido y apagado de una bombilla tradicional.
Encendido y apagado de una luz inteligente.
Encendido y apagado de un ventilador.
Correcta inyección de dependencias mediante constructor.
Funcionamiento del interruptor sin importar el tipo de dispositivo conectado.
Todas las pruebas fueron exitosas.

Conclusión
La solución cumple con el principio Dependency Inversion Principle (DIP), ya que la clase Switch depende de una abstracción y no de una implementación concreta.
Beneficios obtenidos:
Menor acoplamiento entre clases.
Mayor flexibilidad del sistema.
Facilidad para agregar nuevos dispositivos.
Mejor mantenibilidad.
Mayor reutilización de código.
Código más escalable y testeable.
