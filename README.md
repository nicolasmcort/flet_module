# flet_module
El siguiente código representa el módulo de flet que creé para facilitar más el uso de esta herramienta al momento de crear la GUI. Cada widget tiene una posición en la pantalla definida por la clase Widget con los atributos top y left. A algunos se les puede cambiar el alto y el ancho.

En el if __name__ == "__main__", hay una pequeña demostración, aunque muy desorganizada. Con esa probaba las clases. 

Otro dato importante es que la clase PageManager me permite cambiar de página en la app. Flet tiene métodos para esto, pero debido a la estructura que ya había comenzado a crear, era mejor opción escribir un código por separado que oculte los widgets actuales y muestre los de la nueva página.
``` python
import flet as ft # type: ignore

# Clase base para los widgets personalizados
class Widget:
    def __init__(self, value="", left=0, top=0, width=200, height=50):
        self.value = value
        self.left = left
        self.top = top
        self.width = width 
        self.height = height

    def create(self):
        raise NotImplementedError("Debes implementar el método 'create'.")

# Clase para manejar páginas y cambiar entre ellas
class PageManager:
    def __init__(self, page: ft.Page):
        self.page = page
        self.pages = {}  # Diccionario para almacenar las páginas
        self.current_page = None  # Página actual que se muestra

    def add_page(self, name: str, widgets: list):
        """Agrega una nueva página con un nombre y una lista de widgets."""
        self.pages[name] = widgets

    def show_page(self, name: str):
        """Muestra una página específica por su nombre."""
        if name in self.pages:
            self.page.controls.clear()
            self.page.add(ft.Stack(controls=[widget.create() for widget in self.pages[name]]))
            self.page.update()
            self.current_page = name

# Clase para los textos que extiende de Widget
class TextElement(Widget):
    def __init__(self, value="", size=24, left=0, top=0, text_color="black", bg_color="white", width=100, height=50, page=None):
        super().__init__(value, left, top, width, height)
        self.size = size
        self.text_color = text_color
        self.bg_color = bg_color
        self.page = page  # Pasar la referencia de la página
        self.text = ft.Text(value=self.value, size=self.size, color=self.text_color)

        # Crear el Container inicialmente
        self.container = ft.Container(
            content=ft.Column(
                controls=[self.text],
                scroll=ft.ScrollMode.AUTO,
            ),
            left=self.left,
            top=self.top,
            width=self.width,
            height=self.height,
            bgcolor=self.bg_color,
            padding=10,
            border_radius=5,
        )

    def create(self):
        return self.container  # Retornar el container creado

    def update(self, text=None, text_color=None, bg_color=None):
        """Actualiza las propiedades del texto: texto, color del texto, color de fondo"""
        if text is not None:
            self.text.value = text  # Actualizar el texto
        if text_color is not None:
            self.text.color = text_color  # Actualizar color del texto
        if bg_color is not None:
            self.bg_color = bg_color  # Actualizar el color de fondo
            self.container.bgcolor = bg_color  # Actualizar el color de fondo del container directamente

        if self.page:  # Asegurarse de que page esté disponible
            self.page.update()  # Actualizar la página para reflejar el cambio


# Widget DataTable
class Table(Widget):
    def __init__(self, data, left=0, top=0):
        super().__init__("", left, top)
        self.table = ft.DataTable(
            columns=[
                ft.DataColumn(ft.Text("Header 1")),
                ft.DataColumn(ft.Text("Header 2")),
            ],
            rows=[ft.DataRow(cells=[ft.DataCell(ft.Text(cell)) for cell in row]) for row in data],
        )

    def create(self):
        return ft.Container(content=self.table, left=self.left, top=self.top)

# Widget Switch 
class Switch(Widget):
    def __init__(self, value=False, left=0, top=0):
        super().__init__("", left, top)  # "" porque no necesita una etiqueta
        self.state = value  # Estado inicial del Switch (True o False)
        self.switch = ft.Switch(value=value, on_change=self.toggle_state)

    def toggle_state(self, e):
        """Actualiza el estado del Switch."""
        self.state = self.switch.value
        #print(f"Switch cambiado: {self.state}")  # Para verificar en la consola

    def create(self):
        return ft.Container(content=self.switch, left=self.left, top=self.top)


# Widget Dropdown
class Dropdown(Widget):
    def __init__(self, options, left=0, top=0):
        super().__init__("", left, top)
        self.options = options  # Opciones del Dropdown
        self.selected_option = options[0] if options else None  # Opción seleccionada por defecto

        self.dropdown = ft.Dropdown(
            options=[ft.dropdown.Option(option) for option in options],
            on_change=self.update_selected_option
        )

    def update_selected_option(self, e):
        """Actualiza la opción seleccionada en el Dropdown."""
        self.selected_option = e.control.value
        #print(f"Dropdown cambiado: {self.selected_option}")  # Para verificar en la consola

    def create(self):
        return ft.Container(content=self.dropdown, left=self.left, top=self.top)


# Widget RadioGroup
class RadioGroup(Widget):
    def __init__(self, options, left=0, top=0):
        super().__init__("", left, top)
        self.options = options  # Opciones del RadioGroup
        self.selected_option = options[0] if options else None  # Opción seleccionada por defecto

        # Crear el RadioGroup con las opciones
        self.radio_group = ft.RadioGroup(
            content=ft.Column(
                [
                    ft.Radio(
                        value=option,
                        label=option,
                    ) for option in options
                ]
            )
        )

        # Asociar un evento para manejar el cambio de selección
        self.radio_group.on_change = self.update_selected_option

    def update_selected_option(self, e):
        """Actualiza la opción seleccionada en el RadioGroup."""
        self.selected_option = e.control.value
        #print(f"RadioGroup cambiado: {self.selected_option}")  # Para verificar en la consola

    def create(self):
        return ft.Container(content=self.radio_group, left=self.left, top=self.top)


# Widget CheckBox
class CheckBox(Widget):
    def __init__(self, label, left=0, top=0):
        super().__init__(label, left, top)
        self.state = False  # Estado inicial del CheckBox (True o False)
        self.checkbox = ft.Checkbox(label=label, on_change=self.toggle_var)

    def toggle_var(self, e):
        """Actualiza el estado del CheckBox."""
        self.state = self.checkbox.value
        #print(f"CheckBox cambiado: {self.state}")  # Para verificar en la consola

    def create(self):
        return ft.Container(content=self.checkbox, left=self.left, top=self.top)

# Widget Slider
class Slider(Widget):
    def __init__(self, min_value=0, max_value=100, left=0, top=0):
        super().__init__("", left, top)
        self.slider = ft.Slider(min=min_value, max=max_value)

    def create(self):
        return ft.Container(content=self.slider, left=self.left, top=self.top)

# Widget TextArea
""" class TextArea(Widget):
    def __init__(self, left=0, top=0, hint_area="Text Area", on_change=None):
        super().__init__("", left, top)
        self.on_change = on_change
        # Crear el TextField como un área de texto con multiline=True
        self.text_area = ft.TextField(multiline=True, label=hint_area, height=100, on_change=self.on_change)

    def create(self):
        return ft.Container(content=self.text_area, left=self.left, top=self.top)
 """

class TextArea(Widget):
    def __init__(self, left=0, top=0, hint_area="Text Area", on_change=None):
        super().__init__("", left, top)
        self.value = ""  # Para almacenar el texto ingresado
        self.on_change = on_change

        self.text_area = ft.TextField(
            multiline=True,
            label=hint_area,
            height=100,
            on_change=self.on_change,
            on_blur=self.update_text  # Actualizar el valor al perder foco
        )

    def update_text(self, e):
        """Actualiza el valor del TextArea cuando pierde el foco."""
        self.value = e.control.value
        #print(f"Texto actualizado: {self.value}")  # Para depuración

    def create(self):
        return ft.Container(content=self.text_area, left=self.left, top=self.top)

# Widget Rectangle
class Rectangle(Widget):
    def __init__(self, left=0, top=0, width=100, height=50, color="blue"):
        super().__init__("", left, top, width, height)
        self.rectangle = ft.Container(bgcolor=color, top=self.top, left=self.left, width=self.width, height=self.height)

    def create(self):
        return self.rectangle

# Widget Circle
class Circle(Widget):
    def __init__(self, left=0, top=0, diameter=50, color="red"):
        super().__init__("", left, top, diameter, diameter)
        self.circle = ft.Container(bgcolor=color, top=self.top, left=self.left, width=self.width, height=self.height, border_radius=self.width / 2)

    def create(self):
        return self.circle

# Widget ProgressBar
class ProgressBar(Widget):
    def __init__(self, value=0, left=0, top=0):
        super().__init__("", left, top)
        self.progress_bar = ft.ProgressBar(value=value)

    def create(self):
        return ft.Container(content=self.progress_bar, left=self.left, top=self.top)

# Widget Button
class Button(Widget):
    def __init__(self, label, action, left=0, top=0):
        super().__init__("", left, top)
        self.button = ft.ElevatedButton(text=label, on_click=action)

    def create(self):
        return ft.Container(content=self.button, left=self.left, top=self.top)

# Class to select files
class FileSelector: 
    def __init__(self, page: ft.Page):
        self.page = page
        self.selected_file = None  # Almacena la ruta del archivo seleccionado
        self.file_picker = ft.FilePicker(on_result=self._on_file_selected)
        self.page.overlay.append(self.file_picker)  # Agregar el FilePicker al overlay de la página

    def _on_file_selected(self, e: ft.FilePickerResultEvent):
        if e.files:
            self.selected_file = e.files[0].path
            snack_bar = ft.SnackBar(ft.Text(f"Archivo seleccionado: {self.selected_file}"))
            self.page.overlay.append(snack_bar)  # Agregar el SnackBar al overlay
            snack_bar.open = True  # Mostrar el SnackBar
            self.page.update()  # Actualizar la página para que se renderice correctamente

    def select_file(self, e):
        self.file_picker.pick_files(allow_multiple=False)

if __name__ == "__main__":
    # Clase principal de la aplicación
    def main(page: ft.Page):
        # Configuración de la página
        page_manager = PageManager(page)
        page.bgcolor = ft.Colors.BLUE_GREY_500
        page.title = "App con Varios Widgets"

        # Crear los widgets
        text1 = TextElement(value="Aquí van tus preguntas", size=14, left=0, top=125, width=1000, height=250, page=page, text_color=ft.Colors.WHITE, bg_color=ft.Colors.BLUE_GREY_900,)
        text2 = TextElement(value="Text 2", left=500, top=180, text_color="black", bg_color="yellow", width=200, height=100, page=page)

        switch = Switch(left=50, top=320)
        table = Table([["Row 1 Col 1", "Row 1 Col 2"], ["Row 2 Col 1", "Row 2 Col 2"]], left=50, top=400)
        dropdown = Dropdown(["Option 1", "Option 2", "Option 3"], left=400, top=50)
        radio_group = RadioGroup(["Option A", "Option B", "Option C"], left=50, top=600)
        checkbox = CheckBox("Accept Terms", left=200, top=300)
        slider = Slider(left=50, top=800)
        #
        text_area = TextArea(left=50, top=100, hint_area="Ingrese algo :)", on_change=lambda e: add_x())
        rectangle = Rectangle(left=400, top=500, width=300, height=50, color="red")
        circle = Circle(left=1200, top=100, diameter=60, color="orange")
        progress_bar = ProgressBar(left=50, top=1200)

        button_add_x = Button("Add 'x'", lambda e: add_x(), left=50, top=320)
        button_change_color = Button("Change Color", lambda e: change_color(), left=200, top=320)
        button_p1 = Button(label="Ir a Página 2", action=lambda e: page_manager.show_page("page2"), top=150)
        button_p2 = Button(label="Ir a Página 1", action=lambda e: page_manager.show_page("page1"), top=150)      

        # Handle the FileSelector
        file_selector = FileSelector(page)
        
        button_select = Button(label="Select file", action=file_selector.select_file, left=50, top=20)


        # Función para añadir 'x' a ambos textos
        def add_x():
            text1.update(text=text1.text.value + "x")
            text2.update(text=text2.text.value + "x")
            page.update()

        # Función para cambiar el color de fondo y el color del texto de ambos textos
        def change_color():
            text1.update(bg_color="white")
            text2.update(bg_color="white")
            page.update()

        page1_widgets = [text1, radio_group, checkbox, slider, text_area, rectangle, button_add_x, button_p1]
        page2_widgets = [text2, switch, table, dropdown, circle, progress_bar, button_change_color, button_p2]


        # Agregar páginas al PageManager
        page_manager.add_page("page1", page1_widgets)
        page_manager.add_page("page2", page2_widgets)

        # Mostrar la primera página
        page_manager.show_page("page1")



    ft.app(target=main)

```
