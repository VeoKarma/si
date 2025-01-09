import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
import json
import os
import subprocess
import sys
import logging
import matplotlib.pyplot as plt
from tkinter import Toplevel
# Configurar el logger
logging.basicConfig(filename='inventory_manager.log', level=logging.INFO, 
                    format='%(asctime)s - %(levelname)s - %(message)s')

# Intentar instalar openpyxl si no está instalada
try:
    import openpyxl
except ImportError:
    subprocess.check_call([sys.executable, "-m", "pip", "install", "openpyxl"])
    import openpyxl

 from openpyxl import Workbook
 from openpyxl.styles import Font, Alignment, PatternFill, Border, Side
 
 class InventoryManager:
    # Agregar esta función para mostrar gráficos avanzados
    def show_reports(self):
        """Muestra gráficos avanzados del inventario."""
        report_window = Toplevel(self.root)
        report_window.title("Reportes Avanzados")
        report_window.geometry("800x600")

        # Crear un marco para los gráficos
        report_frame = ttk.Frame(report_window, padding=10)
        report_frame.pack(fill=tk.BOTH, expand=True)

        # Generar gráficos
        categories, quantities = self.get_category_data()
        self.plot_bar_chart(categories, quantities, report_frame)
        self.plot_pie_chart(categories, quantities, report_frame)

        # Log de visualización de reportes
        logging.info("Reportes avanzados generados y mostrados.")

    def get_category_data(self):
        """Obtiene los datos de categorías y cantidades."""
        category_data = {}
        for category in self.categories:
            for item in self.tree.get_children(self.categories[category]):
                product_data = self.tree.item(item, 'values')
                category_name = product_data[3]
                quantity = int(product_data[1])
                category_data[category_name] = category_data.get(category_name, 0) + quantity
        categories = list(category_data.keys())
        quantities = list(category_data.values())
        return categories, quantities

    def plot_bar_chart(self, categories, quantities, frame):
        """Genera un gráfico de barras."""
        fig, ax = plt.subplots(figsize=(8, 4))
        ax.bar(categories, quantities, color='skyblue')
        ax.set_title("Cantidad de Productos por Categoría")
        ax.set_xlabel("Categoría")
        ax.set_ylabel("Cantidad")
        ax.grid(axis='y', linestyle='--', alpha=0.7)

        # Integrar gráfico en la ventana de Tkinter
        from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
        canvas = FigureCanvasTkAgg(fig, master=frame)
        canvas.draw()
        canvas.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=True)

    def plot_pie_chart(self, categories, quantities, frame):
        """Genera un gráfico de pastel."""
        fig, ax = plt.subplots(figsize=(6, 6))
        ax.pie(quantities, labels=categories, autopct='%1.1f%%', startangle=90, colors=plt.cm.tab10.colors)
        ax.set_title("Distribución de Productos por Categoría")

        # Integrar gráfico en la ventana de Tkinter
        from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
        canvas = FigureCanvasTkAgg(fig, master=frame)
        canvas.draw()
        canvas.get_tk_widget().pack(side=tk.BOTTOM, fill=tk.BOTH, expand=True)
    def __init__(self, root):
        self.root = root
        self.root.title("Gestión de Inventario")
        self.root.geometry('1280x720')
        self.root.configure(bg="white")

        # Aplicar estilo profesional
        style = ttk.Style()
        style.theme_use('clam')
        style.configure("Treeview", background="#f9f9f9", foreground="black", rowheight=25, fieldbackground="#f9f9f9")
        style.configure("Treeview.Heading", font=("Helvetica", 11, "bold"), background="#4CAF50", foreground="white")
        style.map('Treeview', background=[('selected', '#FF5733')])

        # Configuración del árbol y categorías
        self.categories = {}
        self.tree = None

        # Variables de entrada
        self.entry_name = None
        self.entry_quantity = None
        self.entry_serial = None
        self.entry_category = None
        self.entry_threshold = None

        # Variables de búsqueda
        self.search_var = tk.StringVar()

        # Configurar la interfaz gráfica
        self.init_ui()

        # Intentar cargar inventario desde el archivo JSON
        try:
            self.load_inventory()
        except Exception as e:
            messagebox.showerror("Error", f"Hubo un error al cargar el inventario: {e}")

        # Log de inicialización
        logging.info("Interfaz de usuario inicializada")

    def init_ui(self):
        """Inicializa los elementos de la interfaz gráfica"""
        main_frame = ttk.Frame(self.root, padding=10)
        main_frame.pack(fill=tk.BOTH, expand=True)

        # Frame superior para entradas y botones
        input_frame = ttk.LabelFrame(main_frame, text="Detalles del Producto", padding=10)
        input_frame.pack(fill=tk.X, pady=5)

        # Entradas
        ttk.Label(input_frame, text="Nombre del Producto:").grid(row=0, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_name = ttk.Entry(input_frame)
        self.entry_name.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(input_frame, text="Cantidad:").grid(row=1, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_quantity = ttk.Entry(input_frame)
        self.entry_quantity.grid(row=1, column=1, padx=5, pady=5)

        ttk.Label(input_frame, text="Número de Serie (opcional):").grid(row=2, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_serial = ttk.Entry(input_frame)
        self.entry_serial.grid(row=2, column=1, padx=5, pady=5)

        ttk.Label(input_frame, text="Categoría:").grid(row=3, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_category = ttk.Entry(input_frame)
        self.entry_category.grid(row=3, column=1, padx=5, pady=5)

        ttk.Label(input_frame, text="Umbral de Stock:").grid(row=4, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_threshold = ttk.Entry(input_frame)
        self.entry_threshold.grid(row=4, column=1, padx=5, pady=5)

        # Añadir después de la entrada de "Umbral de Stock"
        ttk.Label(input_frame, text="Ubicación:").grid(row=5, column=0, sticky=tk.W, padx=5, pady=5)
        self.entry_location = ttk.Entry(input_frame)
        self.entry_location.grid(row=5, column=1, padx=5, pady=5)

        # Botones
        button_frame = ttk.Frame(main_frame)
        button_frame.pack(fill=tk.X, pady=5)

        ttk.Button(button_frame, text="Agregar Producto", command=self.add_product).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Eliminar Producto", command=self.delete_product).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Editar Producto", command=self.edit_product).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Guardar Stock", command=self.save_to_json).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Exportar a Excel", command=self.export_to_excel).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Modificar Cantidad", command=self.modify_quantity).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Reportes Gráficos", command=self.show_reports).pack(side=tk.LEFT, padx=5)


        # Barra de búsqueda
        search_frame = ttk.Frame(main_frame)
        search_frame.pack(fill=tk.X, pady=5)
        ttk.Label(search_frame, text="Buscar:").pack(side=tk.LEFT, padx=5)
        search_entry = ttk.Entry(search_frame, textvariable=self.search_var)
        search_entry.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=5)

        # Configurar Treeview
        tree_frame = ttk.Frame(main_frame)
        tree_frame.pack(fill=tk.BOTH, expand=True, pady=5)

        # Añadir la columna "Ubicación" en la configuración del Treeview
        columns = ('Nombre', 'Cantidad', 'Número de Serie', 'Categoría', 'Umbral de Stock', 'Ubicación')
        self.tree = ttk.Treeview(tree_frame, columns=columns, show='headings', selectmode="browse")

        for col in columns:
            self.tree.heading(col, text=col, command=lambda _col=col: self.sort_column(_col, False))
        self.tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

        tree_scroll = ttk.Scrollbar(tree_frame, orient=tk.VERTICAL, command=self.tree.yview)
        self.tree.configure(yscrollcommand=tree_scroll.set)
        tree_scroll.pack(side=tk.RIGHT, fill=tk.Y)

        # Etiqueta de resumen
        self.summary_label = ttk.Label(main_frame, text="Total de productos en inventario: 0", font=("Helvetica", 10, "bold"))
        self.summary_label.pack(pady=5)

        # Enlace para realizar la búsqueda cada vez que se escribe en la barra
        search_entry.bind('<KeyRelease>', self.search_products)

    def load_inventory(self):
        """Carga el inventario desde un archivo JSON"""
        if os.path.exists('stock.json'):
            with open('stock.json', 'r') as file:
                try:
                    products = json.load(file)
                    for product in products:
                        if len(product) == 5:  # Si el producto no tiene ubicación
                            product.append('')  # Agregar ubicación por defecto
                        self._add_to_tree(*product)
                except json.JSONDecodeError:
                    messagebox.showerror("Error", "El archivo JSON está corrupto.")
        else:
            with open('stock.json', 'w') as file:
                json.dump([], file)

        self.update_summary()

    def _add_to_tree(self, name, quantity, serial, category, threshold, location):
        """Agrega un producto al Treeview y a la categoría correspondiente"""
        if category not in self.categories:
            self.categories[category] = self.tree.insert('', tk.END, text=category, open=True)
        item_id = self.tree.insert(self.categories[category], tk.END, values=(name, quantity, serial, category, threshold, location))
        self.check_stock_alert(item_id, quantity, threshold)

    def add_product(self):
        """Agrega un nuevo producto al inventario"""
        name = self.entry_name.get()
        quantity = self.entry_quantity.get()
        serial = self.entry_serial.get()
        category = self.entry_category.get()
        threshold = self.entry_threshold.get()
        location = self.entry_location.get()

        if name and quantity.isdigit() and threshold.isdigit():
            self._add_to_tree(name, quantity, serial, category, threshold, location)
            self.update_summary()

            # Log de adición de producto
            logging.info(f"Producto añadido: {name}, Cantidad: {quantity}, Serial: {serial}, Categoría: {category}, Umbral: {threshold}, Ubicación: {location}")

            # Limpiar campos
            self.entry_name.delete(0, tk.END)
            self.entry_quantity.delete(0, tk.END)
            self.entry_serial.delete(0, tk.END)
            self.entry_category.delete(0, tk.END)
            self.entry_threshold.delete(0, tk.END)
            self.entry_location.delete(0, tk.END)
        else:
            messagebox.showerror("Error", "Por favor, introduce un nombre válido, una cantidad numérica y un umbral de stock numérico.")

    def delete_product(self):
        """Elimina un producto seleccionado del inventario"""
        selected_item = self.tree.selection()
        if selected_item:
            serial = self.tree.item(selected_item, 'values')[2]
            self.tree.delete(selected_item)
            self.update_summary()

            # Log de eliminación de producto
            logging.info(f"Producto eliminado: Serial: {serial}")
        else:
            messagebox.showerror("Error", "Por favor, selecciona un producto para eliminar.")

    def edit_product(self):
        """Edita un producto seleccionado en el inventario"""
        selected_item = self.tree.selection()
        if not selected_item:
            messagebox.showerror("Error", "Por favor, selecciona un producto para editar.")
            return

        item = selected_item[0]
        values = self.tree.item(item, 'values')

        self.modify_window = tk.Toplevel(self.root)
        self.modify_window.title("Editar Producto")
        self.modify_window.geometry('400x300')

        ttk.Label(self.modify_window, text="Nombre:").grid(row=0, column=0, padx=10, pady=10)
        self.modify_name = ttk.Entry(self.modify_window)
        self.modify_name.grid(row=0, column=1, padx=10, pady=10)
        self.modify_name.insert(0, values[0])

        ttk.Label(self.modify_window, text="Cantidad:").grid(row=1, column=0, padx=10, pady=10)
        self.modify_quantity = ttk.Entry(self.modify_window)
        self.modify_quantity.grid(row=1, column=1, padx=10, pady=10)
        self.modify_quantity.insert(0, values[1])

        ttk.Label(self.modify_window, text="Número de Serie:").grid(row=2, column=0, padx=10, pady=10)
        self.modify_serial = ttk.Entry(self.modify_window)
        self.modify_serial.grid(row=2, column=1, padx=10, pady=10)
        self.modify_serial.insert(0, values[2])

        ttk.Label(self.modify_window, text="Categoría:").grid(row=3, column=0, padx=10, pady=10)
        self.modify_category = ttk.Entry(self.modify_window)
        self.modify_category.grid(row=3, column=1, padx=10, pady=10)
        self.modify_category.insert(0, values[3])

        ttk.Label(self.modify_window, text="Umbral de Stock:").grid(row=4, column=0, padx=10, pady=10)
        self.modify_threshold = ttk.Entry(self.modify_window)
        self.modify_threshold.grid(row=4, column=1, padx=10, pady=10)
        self.modify_threshold.insert(0, values[4])

        ttk.Label(self.modify_window, text="Ubicación:").grid(row=5, column=0, padx=10, pady=10)
        self.modify_location = ttk.Entry(self.modify_window)
        self.modify_location.grid(row=5, column=1, padx=10, pady=10)
        self.modify_location.insert(0, values[5])

        ttk.Button(self.modify_window, text="Guardar", command=lambda: self.save_edit(item)).grid(row=6, column=0, columnspan=2, pady=20)

    def save_edit(self, item):
        """Guarda los cambios realizados a un producto"""
        new_name = self.modify_name.get()
        new_quantity = self.modify_quantity.get()
        new_serial = self.modify_serial.get()
        new_category = self.modify_category.get()
        new_threshold = self.modify_threshold.get()
        new_location = self.modify_location.get()

        if new_name and new_quantity.isdigit() and new_threshold.isdigit():
            self.tree.item(item, values=(new_name, new_quantity, new_serial, new_category, new_threshold, new_location))
            self.check_stock_alert(item, new_quantity, new_threshold)
            self.update_summary()
            self.modify_window.destroy()

            # Log de edición de producto
            logging.info(f"Producto editado: {new_name}, Cantidad: {new_quantity}, Serial: {new_serial}, Categoría: {new_category}, Umbral: {new_threshold}, Ubicación: {new_location}")
        else:
            messagebox.showerror("Error", "Por favor, introduce un nombre válido, una cantidad numérica y un umbral de stock numérico.")

    def modify_quantity(self):
        """Modifica la cantidad de un producto seleccionado"""
        selected_item = self.tree.selection()
        if selected_item:
            product_info = self.tree.item(selected_item, 'values')
            self.modify_window = tk.Toplevel(self.root)
            self.modify_window.title("Modificar Cantidad")
            self.modify_window.geometry('300x200')
            
            tk.Label(self.modify_window, text=f"Modificar cantidad para {product_info[0]}").pack(pady=10)
            
            # Campo de entrada para la cantidad
            self.entry_modify_quantity = ttk.Entry(self.modify_window)
            self.entry_modify_quantity.pack(pady=10)
            
            # Botones para sumar o restar cantidad
            ttk.Button(self.modify_window, text="Sumar", command=lambda: self.update_quantity(selected_item, int(self.entry_modify_quantity.get()), "sumar")).pack(side=tk.LEFT, padx=10)
            ttk.Button(self.modify_window, text="Restar", command=lambda: self.update_quantity(selected_item, int(self.entry_modify_quantity.get()), "restar")).pack(side=tk.RIGHT, padx=10)
        else:
            messagebox.showerror("Error", "Por favor, selecciona un producto para modificar la cantidad.")

    def update_quantity(self, item, amount, operation):
        """Actualiza la cantidad de un producto"""
        current_quantity = int(self.tree.item(item, 'values')[1])
        if operation == "sumar":
            new_quantity = current_quantity + amount
        elif operation == "restar":
            new_quantity = current_quantity - amount
        
        if new_quantity < 0:
            messagebox.showerror("Error", "La cantidad no puede ser negativa.")
            return
        
        self.tree.item(item, values=(self.tree.item(item, 'values')[0], new_quantity, self.tree.item(item, 'values')[2], self.tree.item(item, 'values')[3], self.tree.item(item, 'values')[4]))
        self.check_stock_alert(item, new_quantity, self.tree.item(item, 'values')[4])
        self.update_summary()
        self.modify_window.destroy()

    def save_to_json(self):
        """Guarda los datos del inventario en un archivo JSON"""
        products = []
        for category in self.categories:
            for item in self.tree.get_children(self.categories[category]):
                products.append(self.tree.item(item, 'values'))
        with open('stock.json', 'w') as file:
            json.dump(products, file)
        messagebox.showinfo("Guardado", "Datos guardados en stock.json")

        # Log de guardado de datos
        logging.info("Datos guardados en stock.json")

    def export_to_excel(self):
        """Exporta los datos del inventario a un archivo Excel"""
        wb = Workbook()
        ws = wb.active
        ws.title = "Inventario"

        # Definir estilos
        header_font = Font(bold=True, color="FFFFFF")
        header_fill = PatternFill(start_color="4F81BD", end_color="4F81BD", fill_type="solid")
        header_alignment = Alignment(horizontal="center", vertical="center")
        border_style = Border(
            left=Side(style="thin"),
            right=Side(style="thin"),
            top=Side(style="thin"),
            bottom=Side(style="thin")
        )

        # Agregar encabezados estilizados
        headers = ['Nombre', 'Cantidad', 'Número de Serie', 'Categoría', 'Umbral de Stock', 'Ubicación']
        ws.append(headers)
        for col_num, header in enumerate(headers, 1):
            cell = ws.cell(row=1, column=col_num)
            cell.value = header
            cell.font = header_font
            cell.fill = header_fill
            cell.alignment = header_alignment
            cell.border = border_style

        # Agregar los productos del inventario con estilo
        for category in self.categories:
            for item in self.tree.get_children(self.categories[category]):
                values = self.tree.item(item, 'values')
                ws.append(values)

        # Aplicar bordes y alineación a las celdas de datos
        for row in ws.iter_rows(min_row=2, max_row=ws.max_row, min_col=1, max_col=6):
            for cell in row:
                cell.border = border_style
                cell.alignment = Alignment(horizontal="left", vertical="center")

        # Ajustar automáticamente el ancho de las columnas
        for column_cells in ws.columns:
            max_length = 0
            column = column_cells[0].column_letter  # Obtener la letra de la columna
            for cell in column_cells:
                try:
                    if cell.value:
                        max_length = max(max_length, len(str(cell.value)))
                except:
                    pass
            adjusted_width = max_length + 2
            ws.column_dimensions[column].width = adjusted_width

        # Guardar el archivo Excel
        try:
            wb.save("inventario.xlsx")
            messagebox.showinfo("Exportación Exitosa", "Los datos se han exportado a inventario.xlsx")

            # Log de exportación a Excel
            logging.info("Datos exportados a inventario.xlsx")
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo guardar el archivo Excel: {e}")

    def search_products(self, event):
        """Busca productos en el inventario mientras escribes"""
        query = self.search_var.get().lower()

        # Primero, eliminar cualquier etiqueta previa de resaltado
        for item in self.tree.get_children():
            for sub_item in self.tree.get_children(item):
                self.tree.item(sub_item, tags='')

        # Si la barra de búsqueda está vacía, no resaltamos nada
        if query == "":
            return

        # Luego, buscar los productos que coinciden con la consulta
        for item in self.tree.get_children():
            for sub_item in self.tree.get_children(item):
                item_text = " ".join(map(str, self.tree.item(sub_item, 'values'))).lower()
                if query in item_text:
                    self.tree.item(sub_item, tags='match')  # Resaltar

        # Definir el estilo de resaltado para los productos coincidentes
        self.tree.tag_configure('match', background='#FFFF00')  # Fondo amarillo

        # Log de búsqueda
        logging.info(f"Búsqueda realizada: Término: {query}")

    def sort_column(self, col, reverse):
        """Ordena los elementos de una columna en el Treeview"""
        data = [(self.tree.set(k, col), k) for k in self.tree.get_children('')]
        data.sort(reverse=reverse, key=lambda x: x[0])

        for index, (val, k) in enumerate(data):
            self.tree.move(k, '', index)

        self.tree.heading(col, command=lambda: self.sort_column(col, not reverse))

    def update_summary(self):
        """Actualiza el resumen de productos en el inventario"""
        total_products = sum(int(self.tree.item(item, 'values')[1]) for category in self.categories for item in self.tree.get_children(self.categories[category]))
        self.summary_label.config(text=f"Total de productos en inventario: {total_products}")

    def check_stock_alert(self, item_id=None, quantity=None, threshold=None):
        """Verifica el stock y resalta en rojo si está por debajo del umbral"""
        if item_id and quantity and threshold:
            if int(quantity) < int(threshold):
                self.tree.item(item_id, tags='low_stock')
                messagebox.showwarning("Alerta de Stock Bajo", f"El producto {self.tree.item(item_id, 'values')[0]} está por debajo del umbral de stock.")
            else:
                self.tree.item(item_id, tags='')
        else:
            for item in self.tree.get_children():
                for sub_item in self.tree.get_children(item):
                    values = self.tree.item(sub_item, 'values')
                    if int(values[1]) < int(values[4]):
                        self.tree.item(sub_item, tags='low_stock')
                    else:
                        self.tree.item(sub_item, tags='')

        self.tree.tag_configure('low_stock', background='#FF0000')  # Fondo rojo

if __name__ == "__main__":
    root = tk.Tk()
    app = InventoryManager(root)
    root.mainloop()
