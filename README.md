from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.popup import Popup
from kivy.graphics import Color, Rectangle, Line

class CalculadoraApp(BoxLayout):
    historial = []
    valor_manual = None

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.orientation = "vertical"
        self.padding = 10
        self.spacing = 10

        # Fondo de la app (#ECF0F1)
        with self.canvas.before:
            Color(0.93, 0.94, 0.95, 1)  # #ECF0F1
            self.rect = Rectangle(size=self.size, pos=self.pos)
        self.bind(size=self._update_rect, pos=self._update_rect)

        # Diccionario para almacenar los bordes
        self.borders = {}

        self.entradas_interes = {}

        # Visor y botón "Borrar"
        visor_layout = BoxLayout(size_hint=(1, 0.15))
        self.visor = TextInput(
            text="",
            font_size=120,
            readonly=True,
            halign="right",
            background_color=(1, 1, 1, 1),  # #FFFFFF
            foreground_color=(0.2, 0.2, 0.2, 1),  # #333333
            size_hint=(0.75, 1)
        )
        # Agregar borde al visor
        self.visor.border_key = 'visor'
        with self.visor.canvas.after:
            Color(0.33, 0.33, 0.33, 1)  # #555555
            self.borders['visor'] = Line(rectangle=(self.visor.x, self.visor.y, self.visor.width, self.visor.height), width=1)
        self.visor.bind(pos=self._update_border, size=self._update_border)
        visor_layout.add_widget(self.visor)
        borrar_digito_btn = Button(
            text="Borrar",
            font_size=50,
            background_color=(0.17, 0.24, 0.31, 1),  # #2C3E50
            background_normal="",
            color=(1, 1, 1, 1),  # #FFFFFF
            size_hint=(0.25, 1)
        )
        # Agregar borde al botón "Borrar"
        borrar_digito_btn.border_key = 'borrar_digito'
        with borrar_digito_btn.canvas.after:
            Color(0.33, 0.33, 0.33, 1)  # #555555
            self.borders['borrar_digito'] = Line(rectangle=(borrar_digito_btn.x, borrar_digito_btn.y, borrar_digito_btn.width, borrar_digito_btn.height), width=1)
        borrar_digito_btn.bind(pos=self._update_border, size=self._update_border)
        borrar_digito_btn.bind(on_press=self.borrar_ultimo_digito)
        visor_layout.add_widget(borrar_digito_btn)
        self.add_widget(visor_layout)

        # Botones auxiliares (AC, Descuento, Historial, C)
        aux_layout = GridLayout(cols=4, size_hint=(1, 0.12), spacing=10)
        aux_buttons = ["AC", "Descuento", "Historial", "C"]
        for idx, (text, func) in enumerate([
            ("AC", self.borrar_todo),
            ("Descuento", self.calcular_descuento),
            ("Historial", self.mostrar_historial),
            ("C", self.limpiar_visor)
        ]):
            btn = Button(
                text=text,
                font_size=50,
                background_color=(0.17, 0.24, 0.31, 1),  # #2C3E50
                background_normal="",
                color=(1, 1, 1, 1),  # #FFFFFF
                on_press=func
            )
            # Agregar borde al botón
            btn.border_key = f'aux_{idx}'
            with btn.canvas.after:
                Color(0.33, 0.33, 0.33, 1)  # #555555
                self.borders[f'aux_{idx}'] = Line(rectangle=(btn.x, btn.y, btn.width, btn.height), width=1)
            btn.bind(pos=self._update_border, size=self._update_border)
            aux_layout.add_widget(btn)
        self.add_widget(aux_layout)

        # Botones de cuotas (3, 6, 9, 12) con campos de interés
        for cuotas, interes_default in [(3, "0"), (6, "0"), (9, "0"), (12, "5")]:
            cuota_layout = GridLayout(cols=4, size_hint=(1, 0.12), spacing=10)
            cuota_btn = Button(
                text=f"{cuotas} cuotas",
                font_size=70,
                background_color=(0.18, 0.8, 0.44, 1),  # #2ECC71
                background_normal="",
                color=(1, 1, 1, 1),  # #FFFFFF
                on_press=lambda instance, c=cuotas: self.calcular_cuotas(c)
            )
            # Agregar borde al botón
            cuota_btn.border_key = f'cuota_{cuotas}'
            with cuota_btn.canvas.after:
                Color(0.33, 0.33, 0.33, 1)  # #555555
                self.borders[f'cuota_{cuotas}'] = Line(rectangle=(cuota_btn.x, cuota_btn.y, cuota_btn.width, cuota_btn.height), width=1)
            cuota_btn.bind(pos=self._update_border, size=self._update_border)
            cuota_layout.add_widget(cuota_btn)
            cuota_layout.add_widget(Label(size_hint=(0.5, 1)))  # Espacio vacío
            interes_input = TextInput(
                text=interes_default,
                font_size=60,
                size_hint=(0.25, 1),
                background_color=(1, 1, 1, 1),  # #FFFFFF
                foreground_color=(0.2, 0.2, 0.2, 1),  # #333333
                halign="center",  # Centrar horizontalmente
                scroll_from_swipe=False,  # Deshabilitar desplazamiento
                scroll_x=0,  # Evitar desplazamiento horizontal
                scroll_y=0  # Evitar desplazamiento vertical
            )
            # Agregar borde al campo de interés
            interes_input.border_key = f'interes_{cuotas}'
            with interes_input.canvas.after:
                Color(0.33, 0.33, 0.33, 1)  # #555555
                self.borders[f'interes_{cuotas}'] = Line(rectangle=(interes_input.x, interes_input.y, interes_input.width, interes_input.height), width=1)
            interes_input.bind(pos=self._update_border, size=self._update_border)
            self.entradas_interes[cuotas] = interes_input
            cuota_layout.add_widget(interes_input)
            cuota_layout.add_widget(Label(
                text="%",
                font_size=60,
                color=(0.33, 0.33, 0.33, 1),  # #555555
                size_hint=(0.25, 1)
            ))
            self.add_widget(cuota_layout)

        # Botones numéricos y de operaciones
        num_ops = [
            ("7", self.click_boton), ("8", self.click_boton), ("9", self.click_boton), ("/", self.click_boton),
            ("4", self.click_boton), ("5", self.click_boton), ("6", self.click_boton), ("×", self.click_boton),
            ("1", self.click_boton), ("2", self.click_boton), ("3", self.click_boton), ("-", self.click_boton),
            ("0", self.click_boton), ("000", self.agregar_tres_ceros), ("+", self.click_boton), ("=", self.operaciones),
        ]
        for row, i in enumerate(range(0, len(num_ops), 4)):
            row_layout = GridLayout(cols=4, size_hint=(1, 0.12), spacing=10)
            for idx, (text, func) in enumerate(num_ops[i:i+4]):
                if text in ["/", "×", "-", "+", "="]:
                    bg_color = (0.2, 0.6, 0.86, 1)  # #3498DB
                    text_color = (1, 1, 1, 1)  # #FFFFFF
                else:
                    bg_color = (0.84, 0.85, 0.86, 1)  # #D5D8DC
                    text_color = (0.2, 0.2, 0.2, 1)  # #333333
                btn = Button(
                    text=text,
                    font_size=80,
                    background_color=bg_color,
                    background_normal="",
                    color=text_color
                )
                # Agregar borde al botón
                btn.border_key = f'num_op_{row}_{idx}'
                with btn.canvas.after:
                    Color(0.33, 0.33, 0.33, 1)  # #555555
                    self.borders[f'num_op_{row}_{idx}'] = Line(rectangle=(btn.x, btn.y, btn.width, btn.height), width=1)
                btn.bind(pos=self._update_border, size=self._update_border)
                if text == "000":
                    btn.bind(on_press=self.agregar_tres_ceros)
                elif text == "=":
                    btn.bind(on_press=self.operaciones)
                else:
                    btn.bind(on_press=lambda instance, t=text: self.click_boton(t))
                row_layout.add_widget(btn)
            self.add_widget(row_layout)

        # Botones Guardar y Volver
        save_load_layout = GridLayout(cols=2, size_hint=(1, 0.12), spacing=10)
        for idx, (text, func) in enumerate([("GUARDAR", self.guardar_valor_manual), ("VOLVER", self.volver_valor_manual)]):
            btn = Button(
                text=text,
                font_size=50,
                background_color=(0.17, 0.24, 0.31, 1),  # #2C3E50
                background_normal="",
                color=(1, 1, 1, 1),  # #FFFFFF
                on_press=func
            )
            # Agregar borde al botón
            btn.border_key = f'save_load_{idx}'
            with btn.canvas.after:
                Color(0.33, 0.33, 0.33, 1)  # #555555
                self.borders[f'save_load_{idx}'] = Line(rectangle=(btn.x, btn.y, btn.width, btn.height), width=1)
            btn.bind(pos=self._update_border, size=self._update_border)
            save_load_layout.add_widget(btn)
        self.add_widget(save_load_layout)

    def _update_rect(self, instance, value):
        self.rect.pos = self.pos
        self.rect.size = self.size

    def _update_border(self, instance, value):
        # Usar el border_key del widget para encontrar el borde correspondiente
        key = getattr(instance, 'border_key', None)
        if key and key in self.borders:
            self.borders[key].rectangle = (instance.x, instance.y, instance.width, instance.height)

    def formatear_numero(self, valor):
        try:
            num = float(valor)
            if num.is_integer():
                num = int(num)
            return "{:,.0f}".format(num).replace(",", ".")
        except ValueError:
            return valor

    def formatear_visor(self):
        valor = self.visor.text.replace(".", "")
        # Solo formatear si el texto no contiene un guion (para evitar interferir con el formato "valor-porcentaje")
        if "-" not in valor:
            if valor:
                try:
                    num = float(valor)
                    self.visor.text = self.formatear_numero(num)
                except ValueError:
                    pass

    def click_boton(self, valor):
        self.visor.text += str(valor)
        self.formatear_visor()

    def agregar_tres_ceros(self, instance):
        self.visor.text += "000"
        self.formatear_visor()

    def borrar_todo(self, instance):
        self.visor.text = ""
        self.historial.clear()

    def limpiar_visor(self, instance):
        self.visor.text = ""

    def borrar_ultimo_digito(self, instance):
        if self.visor.text:
            self.visor.text = self.visor.text[:-1]

    def operaciones(self, instance):
        try:
            ecuacion = self.visor.text.replace(".", "").replace("×", "*").replace("−", "-")
            resultado = eval(ecuacion)
            resultado_formateado = self.formatear_numero(resultado)
            self.historial.append((ecuacion, resultado))
            self.visor.text = resultado_formateado
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def calcular_porcentaje(self, instance):
        try:
            valor = float(self.visor.text.replace(".", ""))
            resultado = valor / 100
            operacion = f"{self.formatear_numero(valor)} / 100"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def calcular_descuento(self, instance):
        try:
            # Reemplazar el símbolo "−" por un guion simple "-" para manejar ambos formatos
            entrada = self.visor.text.replace(".", "").replace("−", "-")
            if '-' not in entrada:
                self.visor.text = "Error: Ingresa un valor como '1000-10' para aplicar un descuento del 10%"
                return
            partes = entrada.split('-')
            if len(partes) != 2:
                self.visor.text = "Error: Formato inválido. Usa 'valor-porcentaje', ej. '1000-10'"
                return
            valor_str, porcentaje_str = partes
            valor = float(valor_str.strip())
            porcentaje = float(porcentaje_str.strip().replace('%', ''))
            if porcentaje < 0 or porcentaje > 100:
                self.visor.text = "Error: El porcentaje debe estar entre 0 y 100"
                return
            resultado = int(valor * (1 - (porcentaje / 100)))
            operacion = f"{self.formatear_numero(valor)} - {porcentaje}%"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except ValueError as e:
            self.visor.text = "Error: Asegúrate de que el valor y el porcentaje sean números válidos"
        except Exception as e:
            self.visor.text = "Error inesperado: " + str(e)

    def guardar_valor_manual(self, instance):
        self.valor_manual = self.visor.text.replace(".", "")

    def volver_valor_manual(self, instance):
        if self.valor_manual is not None:
            self.visor.text = self.valor_manual
            self.formatear_visor()

    def porc15(self, instance):
        try:
            ecuacion = self.visor.text.replace(".", "")
            resultado = eval(ecuacion)
            resultado_descuento = resultado * 0.15
            resultado -= resultado_descuento
            operacion = f"{self.formatear_numero(eval(ecuacion))} con 15% descuento"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def porc18(self, instance):
        try:
            ecuacion = self.visor.text.replace(".", "")
            resultado = eval(ecuacion)
            resultado_descuento = resultado * 0.18
            resultado -= resultado_descuento
            operacion = f"{self.formatear_numero(eval(ecuacion))} con 18% descuento"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def porc20(self, instance):
        try:
            ecuacion = self.visor.text.replace(".", "")
            resultado = eval(ecuacion)
            resultado_descuento = resultado * 0.20
            resultado -= resultado_descuento
            operacion = f"{self.formatear_numero(eval(ecuacion))} con 20% descuento"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def promo(self, instance):
        try:
            ecuacion = self.visor.text.replace(".", "")
            resultado = eval(ecuacion)
            resultado_descuento = resultado * 0.30
            resultado -= resultado_descuento
            operacion = f"{self.formatear_numero(eval(ecuacion))} con 30% descuento (promo)"
            self.historial.append((operacion, resultado))
            self.visor.text = self.formatear_numero(resultado)
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def calcular_cuotas(self, num_cuotas):
        try:
            ecuacion = self.visor.text.replace(".", "")
            resultado = eval(ecuacion)
            interes_str = self.entradas_interes[num_cuotas].text.strip()
            if interes_str == "":
                porcentaje_interes = 0
            else:
                porcentaje_interes = float(interes_str)
            monto_con_interes = resultado * (1 + porcentaje_interes / 100)
            if monto_con_interes < 0:
                raise ValueError("El monto resultante no puede ser negativo")
            resultado_cuota = monto_con_interes / num_cuotas
            operacion = f"{self.formatear_numero(resultado)} en {num_cuotas} cuotas ({porcentaje_interes}%)"
            self.historial.append((operacion, resultado_cuota))
            self.visor.text = self.formatear_numero(round(resultado_cuota))
        except (SyntaxError, ZeroDivisionError, ValueError) as e:
            self.visor.text = "Error: " + str(e)

    def mostrar_historial(self, instance):
        if not self.historial:
            self.visor.text = "Historial vacío"
            return

        popup = Popup(title="Historial de Operaciones", size_hint=(0.8, 0.8))
        layout = BoxLayout(orientation="vertical")
        
        historial_text = "\n".join(
            f"{i+1}. {expresion} = {self.formatear_numero(round(resultado))}"
            for i, (expresion, resultado) in enumerate(self.historial)
        )
        label = Label(
            text=historial_text,
            font_size=50,
            halign="left",
            valign="top",
            size_hint=(1, 0.9),
            text_size=(None, None)
        )
        label.bind(size=label.setter('text_size'))
        
        cerrar_btn = Button(
            text="Cerrar",
            font_size=50,
            size_hint=(1, 0.1),
            background_color=(0.17, 0.24, 0.31, 1),  # #2C3E50
            background_normal="",
            color=(1, 1, 1, 1),  # #FFFFFF
        )
        # Agregar borde al botón "Cerrar"
        cerrar_btn.border_key = 'cerrar_historial'
        with cerrar_btn.canvas.after:
            Color(0.33, 0.33, 0.33, 1)  # #555555
            self.borders['cerrar_historial'] = Line(rectangle=(cerrar_btn.x, cerrar_btn.y, cerrar_btn.width, cerrar_btn.height), width=1)
        cerrar_btn.bind(pos=self._update_border, size=self._update_border)
        cerrar_btn.bind(on_press=popup.dismiss)
        
        layout.add_widget(label)
        layout.add_widget(cerrar_btn)
        popup.content = layout
        popup.bind(on_dismiss=self.seleccionar_operacion)
        popup.open()

    def seleccionar_operacion(self, instance):
        # Selección no implementada por simplicidad, pero se puede agregar
        pass

class MyApp(App):
    def build(self):
        return CalculadoraApp()

if __name__ == "__main__":
    MyApp().run()
