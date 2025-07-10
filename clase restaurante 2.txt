import json
from collections import deque, namedtuple

Combo = namedtuple("Combo", ["entrada", "plato_principal", "bebida"])

class ItemMenu:
    def __init__(self, nombre: str, precio: float):
        self._nombre = nombre
        self._precio = precio

    def get_nombre(self):
        return self._nombre

    def set_nombre(self, value):
        self._nombre = value

    def get_precio(self):
        return self._precio

    def set_precio(self, value):
        self._precio = value

    def calcular_precio_total(self) -> float:
        return self._precio

    def to_dict(self):
        return {
            "tipo": self.__class__.__name__,
            "nombre": self._nombre,
            "precio": self._precio
        }
    def __str__(self):
        return f"{self._nombre}: ${self._precio:.2f}"

class Bebida(ItemMenu):
    def __init__(self, nombre: str, precio: float, tamaño: str):
        super().__init__(nombre, precio)
        self._tamaño = tamaño

    def get_tamaño(self):
        return self._tamaño

    def set_tamaño(self, value):
        self._tamaño = value

    def to_dict(self):
        data = super().to_dict()
        data["tamaño"] = self._tamaño
        return data

class Entrada(ItemMenu):
    def __init__(self, nombre: str, precio: float, es_compartible: bool):
        super().__init__(nombre, precio)
        self._es_compartible = es_compartible

    def get_es_compartible(self):
        return self._es_compartible

    def set_es_compartible(self, value):
        self._es_compartible = value

    def to_dict(self):
        data = super().to_dict()
        data["es_compartible"] = self._es_compartible
        return data

class PlatoPrincipal(ItemMenu):
    def __init__(self, nombre: str, precio: float, es_vegetariano: bool):
        super().__init__(nombre, precio)
        self._es_vegetariano = es_vegetariano

    def get_es_vegetariano(self):
        return self._es_vegetariano

    def set_es_vegetariano(self, value):
        self._es_vegetariano = value

    def to_dict(self):
        data = super().to_dict()
        data["es_vegetariano"] = self._es_vegetariano
        return data

class Pedido:
    def __init__(self):
        self.items = []

    def agregar_item(self, item: ItemMenu):
        self.items.append(item)

    def calcular_total(self) -> float:
        total = 0.0
        tiene_principal = any(isinstance(item, PlatoPrincipal) for item in self.items)
        for item in self.items:
            if isinstance(item, Bebida) and tiene_principal:
                total += item.get_precio() * 0.85
            else:
                total += item.get_precio()
        return self.aplicar_descuento(total)

    def aplicar_descuento(self, total: float) -> float:
        if len(self.items) >= 5:
            return total * 0.9
        return total

    def mostrar_pedido(self):
        for item in self.items:
            print(item)
        print(f"Total: ${self.calcular_total():.2f}")

    def create_menu(self, menu_name: str, items: list):
        menu_dict = [item.to_dict() for item in items]
        with open(f"{menu_name}.json", "w") as file:
            json.dump(menu_dict, file, indent=4)
        print(f"Menu '{menu_name}' creado y guardado.")

    def add_item_to_menu(self, menu_name: str, item: ItemMenu):
        try:
            with open(f"{menu_name}.json", "r") as file:
                data = json.load(file)
        except FileNotFoundError:
            data = []

        data.append(item.to_dict())

        with open(f"{menu_name}.json", "w") as file:
            json.dump(data, file, indent=4)
        print(f"Ítem agregado a '{menu_name}'.")

    def update_item_in_menu(self, menu_name: str, nombre: str, nuevo_precio: float):
        try:
            with open(f"{menu_name}.json", "r") as file:
                data = json.load(file)
        except FileNotFoundError:
            print(f"Menú '{menu_name}' no encontrado.")
            return
        actualizado = False
        for item in data:
            if item["nombre"] == nombre:
                item["precio"] = nuevo_precio
                actualizado = True
                break
        if actualizado:
            with open(f"{menu_name}.json", "w") as file:
                json.dump(data, file, indent=4)
            print(f"Ítem '{nombre}' actualizado en '{menu_name}'.")
        else:
            print(f"Ítem '{nombre}' no encontrado en '{menu_name}'.")

    def delete_item_from_menu(self, menu_name: str, nombre: str):
        try:
            with open(f"{menu_name}.json", "r") as file:
                data = json.load(file)
        except FileNotFoundError:
            print(f"Menú '{menu_name}' no encontrado.")
            return

        data_filtrada = [item for item in data if item["nombre"] != nombre]

        with open(f"{menu_name}.json", "w") as file:
            json.dump(data_filtrada, file, indent=4)
        print(f"Ítem '{nombre}' eliminado de '{menu_name}'.")

class Pago:
    def __init__(self, pedido: Pedido, metodo: str):
        self.pedido = pedido
        self.metodo = metodo.lower()

    def procesar_pago(self):
        total = self.pedido.calcular_total()
        print(f"Procesando pago de ${total:.2f} mediante {self.metodo.capitalize()}...")
        print("Pago completado")

class GestorPedidos:
    def __init__(self):
        self.cola_pedidos = deque()

    def nuevo_pedido(self, pedido: Pedido):
        self.cola_pedidos.append(pedido)

    def procesar_siguiente(self):
        if self.cola_pedidos:
            pedido = self.cola_pedidos.popleft()
            pedido.mostrar_pedido()
        else:
            print("No hay pedidos en la cola.")

if __name__ == "__main__":

    menu = [
        Bebida("Jugo de Fresa", 3.0, "Grande"),
        Entrada("Buñuelos", 4.5, True),
        PlatoPrincipal("Pollo Apanado", 11.0, False)
    ]

    pedido1 = Pedido()
    pedido1.agregar_item(menu[0])
    pedido1.agregar_item(menu[1])
    pedido1.agregar_item(menu[2])

    gestor_pedidos = GestorPedidos()
    gestor_pedidos.nuevo_pedido(pedido1)

    gestor_pedidos.procesar_siguiente()

    pedido1.create_menu("menu_casero", menu)
    pedido1.add_item_to_menu("menu_casero", Bebida("Limonada", 2.5, "Mediana"))
    pedido1.update_item_in_menu("menu_casero", "Limonada", 2.0)
    pedido1.delete_item_from_menu("menu_casero", "Limonada")

    combo_familiar = Combo(
        entrada=Entrada("Papas a la francesa", 5.0, True),
        plato_principal=PlatoPrincipal("Hamburguesa Vegana", 10.0, True),
        bebida=Bebida("Té Helado", 2.5, "Mediano")
    )

    pedido_combo = Pedido()
    pedido_combo.agregar_item(combo_familiar.entrada)
    pedido_combo.agregar_item(combo_familiar.plato_principal)
    pedido_combo.agregar_item(combo_familiar.bebida)

    print("\nPedido creado a partir de combo familiar")
    pedido_combo.mostrar_pedido()

    pago_combo = Pago(pedido_combo, "efectivo")
    pago_combo.procesar_pago()

    gestor_pedidos.procesar_siguiente()