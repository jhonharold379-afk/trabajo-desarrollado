# trabajo-desarrollado

"""
Proyecto: Sistema Integral de Gestión de Clientes, Servicios y Reservas
Empresa: Software FJ
Curso: Programación - Fase 4

Descripción:
Aplicación de consola que administra clientes, servicios y reservas
utilizando programación orientada a objetos, listas y archivos de texto.
No utiliza bases de datos.

Ejecución:
    python main.py

El programa realiza más de 10 operaciones de prueba, incluye casos válidos
e inválidos y registra eventos y errores en el archivo logs.txt.
"""

from abc import ABC, abstractmethod
from datetime import datetime
import logging


# ------------------------------------------------------------
# 1. CONFIGURACIÓN DEL ARCHIVO DE REGISTRO (LOGS)
# ------------------------------------------------------------

logging.basicConfig(
    filename="logs.txt",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    encoding="utf-8"
)


# ------------------------------------------------------------
# 2. EXCEPCIONES PERSONALIZADAS
# ------------------------------------------------------------

class DatosInvalidosError(Exception):
    """Se genera cuando un dato ingresado no cumple las validaciones."""
    pass


class ClienteNoEncontradoError(Exception):
    """Se genera cuando no existe el cliente solicitado."""
    pass


class ServicioNoDisponibleError(Exception):
    """Se genera cuando un servicio no está disponible."""
    pass


class ReservaInvalidaError(Exception):
    """Se genera cuando una reserva no puede realizarse."""
    pass


# ------------------------------------------------------------
# 3. CLASE ABSTRACTA ENTIDAD
# ------------------------------------------------------------

class Entidad(ABC):
    """Representa una entidad general del sistema."""

    def __init__(self, identificacion):
        if not str(identificacion).strip():
            raise DatosInvalidosError("La identificación no puede estar vacía.")
        self._identificacion = str(identificacion).strip()

    @property
    def identificacion(self):
        return self._identificacion

    @abstractmethod
    def mostrar_informacion(self):
        """Cada entidad debe implementar su forma de mostrar información."""
        pass


# ------------------------------------------------------------
# 4. CLASE CLIENTE
# ------------------------------------------------------------

class Cliente(Entidad):
    """Almacena y valida los datos personales de un cliente."""

    def __init__(self, identificacion, nombre, correo, telefono):
        super().__init__(identificacion)

        if not nombre or not nombre.strip():
            raise DatosInvalidosError("El nombre del cliente es obligatorio.")

        if "@" not in correo or "." not in correo:
            raise DatosInvalidosError("El correo electrónico no es válido.")

        if not str(telefono).isdigit() or len(str(telefono)) < 7:
            raise DatosInvalidosError("El teléfono debe tener mínimo 7 dígitos.")

        self.__nombre = nombre.strip()
        self.__correo = correo.strip()
        self.__telefono = str(telefono)

    @property
    def nombre(self):
        return self.__nombre

    @property
    def correo(self):
        return self.__correo

    @property
    def telefono(self):
        return self.__telefono

    def mostrar_informacion(self):
        return (
            f"ID: {self.identificacion} | Nombre: {self.nombre} | "
            f"Correo: {self.correo} | Teléfono: {self.telefono}"
        )


# ------------------------------------------------------------
# 5. CLASE ABSTRACTA SERVICIO
# ------------------------------------------------------------

class Servicio(ABC):
    """Clase base para todos los servicios ofrecidos por Software FJ."""

    def __init__(self, codigo, nombre, tarifa_base):
        if not codigo or not str(codigo).strip():
            raise DatosInvalidosError("El código del servicio es obligatorio.")

        if not nombre or not nombre.strip():
            raise DatosInvalidosError("El nombre del servicio es obligatorio.")

        if not isinstance(tarifa_base, (int, float)) or tarifa_base <= 0:
            raise DatosInvalidosError("La tarifa debe ser un número mayor que cero.")

        self._codigo = str(codigo).strip()
        self._nombre = nombre.strip()
        self._tarifa_base = float(tarifa_base)
        self._disponible = True

    @property
    def codigo(self):
        return self._codigo

    @property
    def nombre(self):
        return self._nombre

    @property
    def tarifa_base(self):
        return self._tarifa_base

    @property
    def disponible(self):
        return self._disponible

    def cambiar_disponibilidad(self, estado):
        if not isinstance(estado, bool):
            raise DatosInvalidosError("La disponibilidad debe ser True o False.")
        self._disponible = estado
        logging.info(
            "Disponibilidad del servicio %s cambiada a %s",
            self.codigo, estado
        )

    def calcular_costo(self, duracion, impuesto=0.0, descuento=0.0):
        """
        Variante flexible del cálculo de costo:
        permite calcular con impuesto y descuento opcionales.
        """
        if not isinstance(duracion, (int, float)) or duracion <= 0:
            raise DatosInvalidosError("La duración debe ser mayor que cero.")

        if not 0 <= impuesto <= 1:
            raise DatosInvalidosError("El impuesto debe estar entre 0 y 1.")

        if not 0 <= descuento <= 1:
            raise DatosInvalidosError("El descuento debe estar entre 0 y 1.")

        subtotal = self.tarifa_base * duracion
        total = subtotal * (1 - descuento) * (1 + impuesto)
        return round(total, 2)

    @abstractmethod
    def describir(self):
        pass

    @abstractmethod
    def validar_parametros(self, duracion):
        pass


# ------------------------------------------------------------
# 6. SERVICIOS ESPECIALIZADOS (HERENCIA Y POLIMORFISMO)
# ------------------------------------------------------------

class ReservaSala(Servicio):
    """Servicio de reserva de salas por horas."""

    def __init__(self, codigo, nombre, tarifa_base, capacidad):
        super().__init__(codigo, nombre, tarifa_base)
        if not isinstance(capacidad, int) or capacidad <= 0:
            raise DatosInvalidosError("La capacidad debe ser un entero positivo.")
        self.capacidad = capacidad

    def describir(self):
        return (
            f"Reserva de sala: {self.nombre}. "
            f"Capacidad máxima: {self.capacidad} personas."
        )

    def validar_parametros(self, duracion):
        if duracion < 1:
            raise DatosInvalidosError("La sala debe reservarse mínimo una hora.")
        return True


class AlquilerEquipo(Servicio):
    """Servicio de alquiler de equipos."""

    def __init__(self, codigo, nombre, tarifa_base, cantidad):
        super().__init__(codigo, nombre, tarifa_base)
        if not isinstance(cantidad, int) or cantidad <= 0:
            raise DatosInvalidosError("La cantidad de equipos debe ser positiva.")
        self.cantidad = cantidad

    def describir(self):
        return (
            f"Alquiler de equipos: {self.nombre}. "
            f"Unidades disponibles: {self.cantidad}."
        )

    def validar_parametros(self, duracion):
        if duracion < 1:
            raise DatosInvalidosError("El alquiler debe ser mínimo de una hora.")
        return True


class AsesoriaEspecializada(Servicio):
    """Servicio de asesoría especializada."""

    def __init__(self, codigo, nombre, tarifa_base, especialista):
        super().__init__(codigo, nombre, tarifa_base)
        if not especialista or not especialista.strip():
            raise DatosInvalidosError("Debe indicar el nombre del especialista.")
        self.especialista = especialista.strip()

    def describir(self):
        return (
            f"Asesoría especializada: {self.nombre}. "
            f"Especialista: {self.especialista}."
        )

    def validar_parametros(self, duracion):
        if duracion < 1:
            raise DatosInvalidosError("La asesoría debe durar mínimo una hora.")
        return True


# ------------------------------------------------------------
# 7. CLASE RESERVA
# ------------------------------------------------------------

class Reserva:
    """Relaciona un cliente con un servicio y administra su estado."""

    contador = 0

    def __init__(self, cliente, servicio, duracion):
        if not isinstance(cliente, Cliente):
            raise ReservaInvalidaError("Debe seleccionar un cliente válido.")

        if not isinstance(servicio, Servicio):
            raise ReservaInvalidaError("Debe seleccionar un servicio válido.")

        try:
            servicio.validar_parametros(duracion)
            if not servicio.disponible:
                raise ServicioNoDisponibleError(
                    f"El servicio {servicio.nombre} no está disponible."
                )
        except (DatosInvalidosError, ServicioNoDisponibleError) as error:
            raise ReservaInvalidaError(
                "No fue posible crear la reserva."
            ) from error

        Reserva.contador += 1
        self.codigo = Reserva.contador
        self.cliente = cliente
        self.servicio = servicio
        self.duracion = duracion
        self.estado = "Pendiente"
        self.fecha = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        logging.info("Reserva %s creada para el cliente %s",
                     self.codigo, cliente.identificacion)

    def confirmar(self):
        if self.estado != "Pendiente":
            raise ReservaInvalidaError(
                "Solo se pueden confirmar reservas pendientes."
            )
        self.estado = "Confirmada"
        logging.info("Reserva %s confirmada", self.codigo)
        return "Reserva confirmada correctamente."

    def cancelar(self):
        if self.estado == "Cancelada":
            raise ReservaInvalidaError("La reserva ya está cancelada.")
        self.estado = "Cancelada"
        logging.info("Reserva %s cancelada", self.codigo)
        return "Reserva cancelada correctamente."

    def procesar(self, impuesto=0.19, descuento=0.0):
        if self.estado == "Cancelada":
            raise ReservaInvalidaError(
                "No se puede procesar una reserva cancelada."
            )
        return self.servicio.calcular_costo(
            self.duracion, impuesto, descuento
        )

    def mostrar_informacion(self):
        return (
            f"Reserva #{self.codigo} | Cliente: {self.cliente.nombre} | "
            f"Servicio: {self.servicio.nombre} | Duración: {self.duracion} "
            f"hora(s) | Estado: {self.estado} | Fecha: {self.fecha}"
        )


# ------------------------------------------------------------
# 8. CLASE PRINCIPAL DEL SISTEMA
# ------------------------------------------------------------

class SistemaSoftwareFJ:
    """Administra las listas de clientes, servicios y reservas."""

    def __init__(self):
        self.clientes = []
        self.servicios = []
        self.reservas = []

    def registrar_cliente(self, cliente):
        if not isinstance(cliente, Cliente):
            raise DatosInvalidosError("El objeto no corresponde a un cliente.")

        for existente in self.clientes:
            if existente.identificacion == cliente.identificacion:
                raise DatosInvalidosError("Ya existe un cliente con esa identificación.")

        self.clientes.append(cliente)
        logging.info("Cliente registrado: %s", cliente.identificacion)
        return "Cliente registrado correctamente."

    def buscar_cliente(self, identificacion):
        for cliente in self.clientes:
            if cliente.identificacion == str(identificacion):
                return cliente
        raise ClienteNoEncontradoError(
            f"No se encontró el cliente {identificacion}."
        )

    def registrar_servicio(self, servicio):
        if not isinstance(servicio, Servicio):
            raise DatosInvalidosError("El objeto no corresponde a un servicio.")

        for existente in self.servicios:
            if existente.codigo == servicio.codigo:
                raise DatosInvalidosError("Ya existe un servicio con ese código.")

        self.servicios.append(servicio)
        logging.info("Servicio registrado: %s", servicio.codigo)
        return "Servicio registrado correctamente."

    def buscar_servicio(self, codigo):
        for servicio in self.servicios:
            if servicio.codigo == str(codigo):
                return servicio
        raise ServicioNoDisponibleError(
            f"No se encontró el servicio {codigo}."
        )

    def crear_reserva(self, identificacion_cliente, codigo_servicio, duracion):
        cliente = self.buscar_cliente(identificacion_cliente)
        servicio = self.buscar_servicio(codigo_servicio)
        reserva = Reserva(cliente, servicio, duracion)
        self.reservas.append(reserva)
        return reserva

    def mostrar_clientes(self):
        print("\n--- CLIENTES REGISTRADOS ---")
        if not self.clientes:
            print("No hay clientes registrados.")
        for cliente in self.clientes:
            print(cliente.mostrar_informacion())

    def mostrar_servicios(self):
        print("\n--- SERVICIOS REGISTRADOS ---")
        if not self.servicios:
            print("No hay servicios registrados.")
        for servicio in self.servicios:
            # Polimorfismo: cada clase describe su servicio de forma diferente.
            print(servicio.describir())

    def mostrar_reservas(self):
        print("\n--- RESERVAS REGISTRADAS ---")
        if not self.reservas:
            print("No hay reservas registradas.")
        for reserva in self.reservas:
            print(reserva.mostrar_informacion())


# ------------------------------------------------------------
# 9. DEMOSTRACIÓN DE OPERACIONES Y MANEJO DE EXCEPCIONES
# ------------------------------------------------------------

def ejecutar_pruebas():
    sistema = SistemaSoftwareFJ()

    print("=" * 65)
    print("       SISTEMA INTEGRAL DE GESTIÓN - SOFTWARE FJ")
    print("=" * 65)

    print("\nOPERACIÓN 1: Registrar primer cliente.")
    try:
        cliente1 = Cliente("1001", "Carlos Pérez",
                          "carlos@gmail.com", "3101234567")
        print(sistema.registrar_cliente(cliente1))
    except DatosInvalidosError as error:
        logging.error("Error en operación 1: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 2: Registrar segundo cliente.")
    try:
        cliente2 = Cliente("1002", "María Gómez",
                          "maria@gmail.com", "3111234567")
        print(sistema.registrar_cliente(cliente2))
    except DatosInvalidosError as error:
        logging.error("Error en operación 2: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 3: Intentar registrar un cliente con correo inválido.")
    try:
        cliente_invalido = Cliente("1003", "Pedro López",
                                   "correo_incorrecto", "3121234567")
        sistema.registrar_cliente(cliente_invalido)
    except DatosInvalidosError as error:
        logging.error("Error controlado en operación 3: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 4: Intentar registrar un cliente duplicado.")
    try:
        cliente_duplicado = Cliente("1001", "Otro Nombre",
                                    "otro@gmail.com", "3131234567")
        sistema.registrar_cliente(cliente_duplicado)
    except DatosInvalidosError as error:
        logging.error("Error controlado en operación 4: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 5: Registrar servicio de sala.")
    try:
        sala = ReservaSala("S01", "Sala de reuniones", 50000, 20)
        print(sistema.registrar_servicio(sala))
    except DatosInvalidosError as error:
        logging.error("Error en operación 5: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 6: Registrar servicio de alquiler de equipos.")
    try:
        equipo = AlquilerEquipo("E01", "Proyector", 30000, 5)
        print(sistema.registrar_servicio(equipo))
    except DatosInvalidosError as error:
        logging.error("Error en operación 6: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 7: Registrar servicio de asesoría.")
    try:
        asesoria = AsesoriaEspecializada(
            "A01", "Asesoría de software", 80000, "Ingeniero Andrés"
        )
        print(sistema.registrar_servicio(asesoria))
    except DatosInvalidosError as error:
        logging.error("Error en operación 7: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 8: Intentar crear un servicio con tarifa inválida.")
    try:
        servicio_invalido = ReservaSala("S02", "Sala inválida", -1000, 10)
        sistema.registrar_servicio(servicio_invalido)
    except DatosInvalidosError as error:
        logging.error("Error controlado en operación 8: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 9: Crear una reserva correcta.")
    try:
        reserva1 = sistema.crear_reserva("1001", "S01", 2)
        print(reserva1.mostrar_informacion())
    except (ClienteNoEncontradoError, ServicioNoDisponibleError,
            ReservaInvalidaError) as error:
        logging.error("Error en operación 9: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 10: Confirmar la reserva.")
    try:
        print(reserva1.confirmar())
    except ReservaInvalidaError as error:
        logging.error("Error en operación 10: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 11: Calcular costo con impuesto.")
    try:
        total = reserva1.procesar(impuesto=0.19)
        print(f"Costo total de la reserva: ${total:,.2f}")
    except (DatosInvalidosError, ReservaInvalidaError) as error:
        logging.error("Error en operación 11: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 12: Crear reserva con cliente inexistente.")
    try:
        sistema.crear_reserva("9999", "S01", 1)
    except ClienteNoEncontradoError as error:
        logging.error("Error controlado en operación 12: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 13: Intentar reservar un servicio inexistente.")
    try:
        sistema.crear_reserva("1001", "X99", 1)
    except ServicioNoDisponibleError as error:
        logging.error("Error controlado en operación 13: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 14: Crear reserva con duración inválida.")
    try:
        sistema.crear_reserva("1001", "S01", 0)
    except ReservaInvalidaError as error:
        logging.error("Error controlado en operación 14: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 15: Cancelar la reserva.")
    try:
        print(reserva1.cancelar())
    except ReservaInvalidaError as error:
        logging.error("Error en operación 15: %s", error)
        print("Error:", error)

    print("\nOPERACIÓN 16: Intentar procesar una reserva cancelada.")
    try:
        reserva1.procesar()
    except ReservaInvalidaError as error:
        logging.error("Error controlado en operación 16: %s", error)
        print("Error controlado:", error)

    print("\nOPERACIÓN 17: Demostrar servicio no disponible.")
    try:
        equipo.cambiar_disponibilidad(False)
        sistema.crear_reserva("1002", "E01", 2)
    except ReservaInvalidaError as error:
        logging.error("Error controlado en operación 17: %s", error)
        print("Error controlado:", error)
    finally:
        print("Bloque finally ejecutado: finalizó la prueba de disponibilidad.")

    print("\nOPERACIÓN 18: Demostrar cálculo con descuento.")
    try:
        costo = asesoria.calcular_costo(2, impuesto=0.19, descuento=0.10)
        print(f"Costo de asesoría con 10% de descuento: ${costo:,.2f}")
    except DatosInvalidosError as error:
        logging.error("Error en operación 18: %s", error)
        print("Error:", error)
    else:
        print("El cálculo se realizó correctamente.")

    # Mostrar información final del sistema.
    sistema.mostrar_clientes()
    sistema.mostrar_servicios()
    sistema.mostrar_reservas()

    print("\n" + "=" * 65)
    print("PRUEBAS FINALIZADAS.")
    print("Los eventos y errores quedaron registrados en logs.txt.")
    print("=" * 65)


# ------------------------------------------------------------
# 10. PUNTO DE INICIO DEL PROGRAMA
# ------------------------------------------------------------

if __name__ == "__main__":
    try:
        ejecutar_pruebas()
    except Exception as error:
        # Captura final para evitar que un error inesperado cierre el programa.
        logging.exception("Error inesperado en la ejecución principal.")
        print("Se presentó un error inesperado:", error)
    finally:
        logging.info("Ejecución del programa finalizada.")
        print("\nPrograma finalizado.")
