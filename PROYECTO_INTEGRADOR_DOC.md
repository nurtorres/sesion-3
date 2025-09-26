
# Proyecto Integrador – MVP (Tkinter)
Guía técnica y de arquitectura para desarrolladores/as Python.

> **Resumen:** Este proyecto es una app de escritorio hecha con **Tkinter** que centraliza varias ventanas (Home, Formulario, Lista CRUD, Tabla con `Treeview` y Canvas) desde una ventana principal. El módulo de entrada orquesta la navegación invocando funciones `open_win_*` definidas en el paquete `app`.

---

## 1) Estructura prevista del proyecto

```
proyecto/
├─ main.py                     # Punto de entrada (este archivo)
└─ app/
   ├─ __init__.py
   ├─ win_home.py              # def open_win_home(root): ...
   ├─ win_form.py              # def open_win_form(root): ...
   ├─ win_list.py              # def open_win_list(root): ...
   ├─ win_table.py             # def open_win_table(root): ...
   └─ win_canvas.py            # def open_win_canvas(root): ...
```

> **Nota:** En este documento solo se recibió el contenido de `main.py`. Las descripciones de los módulos `app.win_*` se infieren a partir de sus nombres y convenciones comunes de Tkinter.

---

## 2) Requisitos

- **Python 3.8+** (recomendado 3.10 o superior)
- **Tkinter** (incluido por defecto en la mayoría de instalaciones de Python en Windows/macOS. En algunas distros Linux puede requerir `sudo apt-get install python3-tk`).
- No se observan dependencias de terceros más allá de la biblioteca estándar.

---

## 3) Ejecución

Desde la raíz del proyecto:

```bash
python main.py
```

La ventana principal se abre con el título **"Proyecto Integrador - MVP"** y tamaño inicial **420x340**.

---

## 4) Walkthrough del código (`main.py`)

```python
import tkinter as tk
from tkinter import ttk
from app.win_home import open_win_home
from app.win_form import open_win_form
from app.win_list import open_win_list
from app.win_table import open_win_table
from app.win_canvas import open_win_canvas
```

- **`tkinter`/`ttk`**: construcción de UI. `ttk` provee widgets con estilo nativo.
- **Importaciones `open_win_*`**: funciones de fábrica/creación de ventanas (p.ej. `tk.Toplevel`) que reciben la raíz (`root`) para gestionar jerarquía, foco y modality (si aplica).

```python
def main():
    root = tk.Tk()
    root.title("Proyecto Integrador - MVP")
    root.geometry("420x340")

    frame = ttk.Frame(root, padding=16)
    frame.pack(fill="both", expand=True)
```
- Se crea la **raíz** de Tk y un `Frame` contenedor con padding para alojar los controles.

```python
    ttk.Label(frame, text="Aplicación Demo (tkinter)", font=("Segoe UI", 12, "bold")).pack(pady=(0, 12))
```
- Encabezado visual de la app.

```python
    ttk.Button(frame, text="1) Home / Bienvenida", command=lambda: open_win_home(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="2) Formulario",        command=lambda: open_win_form(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="3) Lista (CRUD básico)", command=lambda: open_win_list(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="4) Tabla (Treeview)",  command=lambda: open_win_table(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="5) Canvas (Dibujo)",   command=lambda: open_win_canvas(root)).pack(pady=4, fill="x")
```
- Cada botón **desacopla** la lógica de UI: la ventana principal solo **invoca** a la función correspondiente del submódulo, siguiendo el patrón _command callbacks_.  
- **`lambda: open_win_xxx(root)`** asegura que el `root` se pase en tiempo de click.

```python
    ttk.Separator(frame).pack(pady=6, fill="x")
    ttk.Button(frame, text="Salir", command=root.destroy).pack(pady=6)
```
- Separador visual y botón para **cerrar la app** invocando `root.destroy()`.

```python
    root.mainloop()
```
- Bucle de eventos de Tk.

```python
if __name__ == "__main__":
    main()
```
- Ejecuta `main()` si el módulo es lanzado directamente (no importado).

---

## 5) Contratos esperados de las ventanas (`app.win_*`)

Aunque no se recibieron los archivos, un contrato coherente sería:

- Cada módulo define una función pública `open_win_*(root) -> tk.Toplevel|None` que:
  - Crea una `tk.Toplevel(root)` para su propia UI.
  - Configura título, tamaño y widgets.
  - Maneja el **ciclo de vida** (p.ej., `protocol("WM_DELETE_WINDOW", ...)`).
  - Puede interactuar con **estado compartido** (si existiera) mediante inyección o patrones (ver §6).

**Sugerencias por pantalla:**
- **`win_home`**: texto de bienvenida, versión, atajos.
- **`win_form`**: campos `ttk.Entry`, validación, botones Guardar/Cancelar.
- **`win_list`**: lista con operaciones CRUD simples (alta/baja/modificación en memoria).
- **`win_table`**: `ttk.Treeview` con columnas, encabezados, sort al click, scrollbars.
- **`win_canvas`**: `tk.Canvas` con herramientas de dibujo (línea, rectángulo, limpiar).

---

## 6) Arquitectura y patrones recomendados

- **Separación por módulo (ya aplicada):** cada ventana en su archivo. Facilita pruebas y mantenimiento.
- **Patrón Controlador / Servicio:** encapsular lógica de negocio en clases o funciones fuera de la UI.
- **Estado compartido** (opcional):
  - Definir un objeto `AppState` (p.ej., `dataclasses.dataclass`) que viva en `main` y se pase a cada `open_win_*` si se requiere compartir datos (lista de elementos, configuración, etc.).
- **Mensajería entre ventanas:**
  - Señales simples a través de callbacks: pasar funciones `on_saved`, `on_deleted`, etc.
  - O usar `event_generate`/`bind` con eventos personalizados `<<ItemSaved>>`.

---

## 7) Gestión de errores y UX

- En cada `open_win_*`, envolver operaciones de E/S en `try/except` y notificar con `tk.messagebox`.
- Deshabilitar botones mientras se procesa (`state="disabled"`) y reactivarlos al terminar.
- Validación de formularios con `register` y `validatecommand` (Tk).

---

## 8) Estilo y consistencia (UI)

- Mantener **tipografías** coherentes: `("Segoe UI", 10)` en Windows, `("SF Pro Text", 12)` en macOS (opcional).
- Usar **`ttk.Style()`** para un tema consistente (p.ej., `"clam"`, `"alt"`, `"vista"`).
- Margen vertical uniforme (`pady=4/6/12`) y `fill="x"` para botones primarios.

---

## 9) Extender con una nueva ventana

1. Crear `app/win_about.py`:
   ```python
   import tkinter as tk
   from tkinter import ttk

   def open_win_about(root):
       win = tk.Toplevel(root)
       win.title("Acerca de")
       ttk.Label(win, text="Proyecto Integrador – v1.0").pack(padx=16, pady=16)
       ttk.Button(win, text="Cerrar", command=win.destroy).pack(pady=8)
       return win
   ```
2. Importar en `main.py`:
   ```python
   from app.win_about import open_win_about
   ```
3. Añadir botón:
   ```python
   ttk.Button(frame, text="6) Acerca de", command=lambda: open_win_about(root)).pack(pady=4, fill="x")
   ```

---

## 10) Pruebas rápidas (manuales)

- Lanzar la app y abrir cada ventana.
- Intentar cerrar la raíz con ventanas hijas abiertas (confirmar que se cierran en cascada o que la raíz gestiona el cierre).
- Verificar que `win_list` mantiene el estado si se vuelve a abrir (si ese es el comportamiento deseado).

---

## 11) Buenas prácticas de empaquetado

- Añadir `pyproject.toml` o `requirements.txt` si se incorporan dependencias futuras.
- Configurar **`__all__`** en `app/__init__.py` si se exportan APIs.
- Usar **PEP 8**: nombres de funciones en `snake_case`, constantes en `UPPER_SNAKE_CASE`.

---

## 12) Limitaciones conocidas (con la info recibida)

- No se incluyen los archivos `app/win_*.py`. La documentación sobre esas ventanas es inferida.
- No se define persistencia (BD/archivo); se asume estado en memoria.
- No hay internacionalización (i18n) ni pruebas automatizadas.

---

## 13) Snippet del `main.py` anotado

```python
def main():
    root = tk.Tk()                                # Crea la ventana raíz
    root.title("Proyecto Integrador - MVP")       # Título
    root.geometry("420x340")                      # Tamaño inicial (px ancho x alto)

    frame = ttk.Frame(root, padding=16)           # Contenedor principal con padding
    frame.pack(fill="both", expand=True)          # Expandir para ocupar todo el espacio

    ttk.Label(frame, text="Aplicación Demo (tkinter)", font=("Segoe UI", 12, "bold")).pack(pady=(0, 12))

    # Navegación a ventanas hijas (desacopladas)
    ttk.Button(frame, text="1) Home / Bienvenida",   command=lambda: open_win_home(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="2) Formulario",          command=lambda: open_win_form(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="3) Lista (CRUD básico)", command=lambda: open_win_list(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="4) Tabla (Treeview)",    command=lambda: open_win_table(root)).pack(pady=4, fill="x")
    ttk.Button(frame, text="5) Canvas (Dibujo)",     command=lambda: open_win_canvas(root)).pack(pady=4, fill="x")

    ttk.Separator(frame).pack(pady=6, fill="x")     # Separador visual
    ttk.Button(frame, text="Salir", command=root.destroy).pack(pady=6)  # Cerrar app

    root.mainloop()                                 # Bucle de eventos de Tk
```

---

## 14) Roadmap sugerido
- Añadir **persistencia** (SQLite con `sqlite3` del stdlib o `peewee`).
- Incorporar **tests** con `pytest` y pruebas de UI mínimas (p.ej., `pytest-tkinter`/`pytest-xvfb` en CI).
- Sistema de **temas** (oscuro/claro) con `ttk.Style()`.
- Manejo de **configuración** en `~/.config/proyecto/config.toml`.

---

## 15) Licencia y créditos
- Definir una licencia (MIT/BSD/Apache-2.0) y un archivo `LICENSE`.
- Añadir `README.md` con badges y capturas de pantalla cuando estén las ventanas finales.

---

**Contacto y mantenimiento**  
Dejar en `README.md` la forma de contribuir y un CHANGELOG con SemVer.


---

# Módulo: `app/win_canvas.py`

Este módulo define la función `open_win_canvas`, responsable de crear y mostrar una ventana secundaria con un **Canvas** para realizar dibujos simples de ejemplo.

## Código

```python
import tkinter as tk
from tkinter import ttk

def open_win_canvas(parent: tk.Tk):
    win = tk.Toplevel(parent)
    win.title("Canvas (Dibujo)")
    win.geometry("480x340")

    frm = ttk.Frame(win, padding=12)
    frm.pack(fill="both", expand=True)

    canvas = tk.Canvas(frm, width=440, height=240, bg="white")
    canvas.pack()

    # Dibujos de ejemplo
    canvas.create_rectangle(20, 20, 120, 80, outline="black", width=2)
    canvas.create_oval(150, 20, 220, 90, fill="lightblue")
    canvas.create_line(240, 30, 360, 90, width=3)
    canvas.create_text(220, 140, text="¡Hola Canvas!", font=("Segoe UI", 12, "bold"))

    ttk.Button(frm, text="Cerrar", command=win.destroy).pack(pady=8, anchor="e")
```

## Explicación paso a paso

1. **Creación de ventana secundaria**
   - `win = tk.Toplevel(parent)` crea una nueva ventana hija de la raíz (`parent`).  
   - Se asigna título `"Canvas (Dibujo)"` y tamaño inicial `480x340` píxeles.

2. **Frame contenedor**
   - `ttk.Frame(win, padding=12)` genera un marco con padding.  
   - Se usa `.pack(fill="both", expand=True)` para que ocupe todo el espacio disponible.

3. **Widget Canvas**
   - `tk.Canvas(frm, width=440, height=240, bg="white")` define un área de dibujo blanca.  
   - Se coloca en el `Frame` con `.pack()`.

4. **Elementos dibujados en el Canvas**
   - **Rectángulo:** `canvas.create_rectangle(20, 20, 120, 80, outline="black", width=2)` dibuja un rectángulo con borde negro.
   - **Óvalo:** `canvas.create_oval(150, 20, 220, 90, fill="lightblue")` genera un círculo/óvalo relleno azul claro.
   - **Línea:** `canvas.create_line(240, 30, 360, 90, width=3)` traza una línea oblicua con grosor 3px.
   - **Texto:** `canvas.create_text(220, 140, text="¡Hola Canvas!", font=("Segoe UI", 12, "bold"))` escribe un texto en la posición indicada.

5. **Botón de cierre**
   - `ttk.Button(frm, text="Cerrar", command=win.destroy)` agrega un botón alineado a la derecha (`anchor="e"`).  
   - Al pulsarlo, se destruye la ventana `Toplevel`.

## Observaciones y buenas prácticas

- Este módulo **no guarda estado**, se limita a mostrar un lienzo con figuras estáticas.  
- Se puede extender con eventos del ratón (`<Button-1>`, `<B1-Motion>`) para permitir **dibujo interactivo**.  
- `Canvas` soporta coordenadas absolutas (px), capas (`tag_raise`, `tag_lower`) y animaciones (`after`).

## Posibles extensiones

- Herramienta para dibujar líneas libres con el ratón.  
- Botón para limpiar el lienzo (`canvas.delete("all")`).  
- Guardado del contenido del canvas como imagen (`postscript` exportado).  
- Diferentes colores, grosores y tipografías configurables desde la UI.

---

---

# Módulo: `app/win_form.py`

Este módulo implementa un formulario simple que permite al usuario ingresar un **nombre** y una **edad**, y guardar esos datos en un archivo de texto.

## Código

```python
import tkinter as tk
from tkinter import ttk, filedialog, messagebox

def open_win_form(parent: tk.Tk):
    win = tk.Toplevel(parent)
    win.title("Formulario")
    win.geometry("420x260")
    frm = ttk.Frame(win, padding=16)
    frm.pack(fill="both", expand=True)

    ttk.Label(frm, text="Nombre:").grid(row=0, column=0, sticky="w")
    ent_nombre = ttk.Entry(frm, width=28)
    ent_nombre.grid(row=0, column=1, pady=4)

    ttk.Label(frm, text="Edad:").grid(row=1, column=0, sticky="w")
    ent_edad = ttk.Entry(frm, width=10)
    ent_edad.grid(row=1, column=1, sticky="w", pady=4)

    def validar_y_guardar():
        nombre = ent_nombre.get().strip()
        edad_txt = ent_edad.get().strip()
        if not nombre:
            messagebox.showerror("Error", "El nombre es requerido.")
            return
        if not edad_txt.isdigit():
            messagebox.showerror("Error", "La edad debe ser un número entero.")
            return
        ruta = filedialog.asksaveasfilename(defaultextension=".txt",
                                            filetypes=[("Texto", "*.txt")])
        if ruta:
            with open(ruta, "w", encoding="utf-8") as f:
                f.write(f"Nombre: {nombre}\nEdad: {edad_txt}\n")
            messagebox.showinfo("OK", "Datos guardados.")

    ttk.Button(frm, text="Guardar", command=validar_y_guardar)\
        .grid(row=3, column=0, pady=12)
    ttk.Button(frm, text="Cerrar", command=win.destroy)\
        .grid(row=3, column=1, sticky="e", pady=12)
```

## Explicación paso a paso

1. **Creación de la ventana**
   - `tk.Toplevel(parent)` abre una ventana hija con título `"Formulario"` y tamaño `420x260`.

2. **Formulario con `ttk.Frame`**
   - Se usan `Label` y `Entry` dispuestos con **grid**:
     - Fila 0: "Nombre" y caja de texto (`ent_nombre`).
     - Fila 1: "Edad" y caja de texto (`ent_edad`).

3. **Función interna `validar_y_guardar`**
   - Recupera valores de los campos.
   - Valida:
     - Nombre no vacío.
     - Edad es un número entero (uso de `isdigit()`).
   - Si la validación falla, muestra `messagebox.showerror`.
   - Si pasa la validación:
     - Abre diálogo `filedialog.asksaveasfilename` para elegir dónde guardar.
     - Escribe el contenido en un archivo `.txt`.
     - Confirma con `messagebox.showinfo("OK", "Datos guardados.")`.

4. **Botones**
   - **Guardar:** ejecuta `validar_y_guardar`.
   - **Cerrar:** destruye la ventana con `win.destroy`.

## Observaciones y buenas prácticas

- Validación mínima: no contempla límites de edad (e.g., 0–120).
- El guardado es **sin formato estructurado** (solo texto plano). Para interoperabilidad futura se recomienda JSON o CSV.
- El uso de `with open(...)` asegura cierre automático del archivo.
- `grid()` se usa aquí en lugar de `pack()`, adecuado para formularios tabulares.

## Posibles extensiones

- Validaciones adicionales (edad mínima/máxima, nombre sin caracteres inválidos).
- Añadir más campos (correo, teléfono, etc.).
- Exportar a **CSV** o **JSON** en lugar de `.txt`.
- Conectar con una base de datos o backend REST.
- Incorporar pruebas unitarias con `unittest.mock` para simular `filedialog` y `messagebox`.

---

---

# Módulo: `app/win_home.py`

Este módulo implementa la ventana de **inicio / bienvenida** de la aplicación.  
Su objetivo es dar un mensaje introductorio y probar la interacción con cuadros de diálogo (`messagebox`).

## Código

```python
import tkinter as tk
from tkinter import ttk, messagebox

def open_win_home(parent: tk.Tk):
    win = tk.Toplevel(parent)
    win.title("Home / Bienvenida")
    win.geometry("360x220")
    frm = ttk.Frame(win, padding=16)
    frm.pack(fill="both", expand=True)

    ttk.Label(frm, text="¡Bienvenid@s!", font=("Segoe UI", 11, "bold")).pack(pady=(0, 8))
    ttk.Label(frm, text="Explora las ventanas desde la pantalla principal.").pack(pady=(0, 12))
    ttk.Button(frm, text="Mostrar mensaje",
               command=lambda: messagebox.showinfo("Info", "¡Equipo listo!")).pack()
    ttk.Button(frm, text="Cerrar", command=win.destroy).pack(pady=8)
```

## Explicación paso a paso

1. **Creación de ventana hija**
   - `tk.Toplevel(parent)` genera una ventana secundaria con título `"Home / Bienvenida"` y tamaño `360x220`.

2. **Frame principal**
   - Se añade un `ttk.Frame` con padding para contener los widgets.

3. **Etiquetas (`Label`)**
   - `"¡Bienvenid@s!"` en negritas como encabezado.
   - Texto secundario que invita a explorar las demás ventanas.

4. **Botones**
   - **Mostrar mensaje:** al hacer clic ejecuta `messagebox.showinfo("Info", "¡Equipo listo!")`.
   - **Cerrar:** destruye la ventana con `win.destroy`.

## Observaciones

- Es una ventana simple de bienvenida y prueba de notificaciones.
- Ideal para mostrar mensajes de estado del sistema o instrucciones iniciales.

## Posibles extensiones

- Mostrar **información dinámica** (ej. versión de la app, fecha/hora actual).
- Añadir un botón para **abrir documentación** o un **enlace web**.
- Incluir un logo o imagen con `tk.PhotoImage`.

---

---

# Módulo: `app/win_list.py`

Este módulo implementa una ventana con una **lista básica (Listbox)** y operaciones CRUD simplificadas: agregar, eliminar y limpiar elementos.

## Código

```python
import tkinter as tk
from tkinter import ttk, messagebox

def open_win_list(parent: tk.Tk):
    win = tk.Toplevel(parent)
    win.title("Lista (CRUD básico)")
    win.geometry("420x300")

    frm = ttk.Frame(win, padding=12)
    frm.pack(fill="both", expand=True)

    lb = tk.Listbox(frm, height=10)
    lb.grid(row=0, column=0, rowspan=4, sticky="nsew", padx=(0, 8))
    frm.columnconfigure(0, weight=1)
    frm.rowconfigure(0, weight=1)

    ent_item = ttk.Entry(frm)
    ent_item.grid(row=0, column=1, sticky="ew")

    def agregar():
        v = ent_item.get().strip()
        if v:
            lb.insert("end", v)
            ent_item.delete(0, "end")
        else:
            messagebox.showwarning("Aviso", "Escribe un texto para agregar.")

    def eliminar():
        sel = lb.curselection()
        if sel:
            lb.delete(sel[0])

    def limpiar():
        lb.delete(0, "end")

    ttk.Button(frm, text="Agregar", command=agregar).grid(row=1, column=1, sticky="ew", pady=4)
    ttk.Button(frm, text="Eliminar seleccionado", command=eliminar).grid(row=2, column=1, sticky="ew", pady=4)
    ttk.Button(frm, text="Limpiar", command=limpiar).grid(row=3, column=1, sticky="ew", pady=4)

    ttk.Button(frm, text="Cerrar", command=win.destroy).grid(row=4, column=0, columnspan=2, pady=10, sticky="e")
```

## Explicación paso a paso

1. **Ventana secundaria**
   - `tk.Toplevel(parent)` crea la ventana con título `"Lista (CRUD básico)"` y tamaño inicial `420x300`.

2. **Frame principal**
   - `ttk.Frame` con padding, empaquetado en `expand=True` para ocupar el área completa.

3. **Listbox**
   - `tk.Listbox(frm, height=10)` crea la lista.
   - Disposición en `grid` con `rowspan=4` para alinear con los botones.
   - Se configura `frm.columnconfigure` y `frm.rowconfigure` para que se expanda correctamente.

4. **Campo de entrada**
   - `ttk.Entry` para escribir un nuevo ítem a agregar.

5. **Funciones internas**
   - **`agregar()`**: inserta el texto del entry en la lista. Si está vacío, muestra advertencia.
   - **`eliminar()`**: borra el elemento actualmente seleccionado (`curselection`).
   - **`limpiar()`**: elimina todos los elementos (`delete(0, "end")`).

6. **Botones CRUD**
   - `"Agregar"` llama a `agregar`.
   - `"Eliminar seleccionado"` llama a `eliminar`.
   - `"Limpiar"` borra toda la lista.

7. **Botón Cerrar**
   - `command=win.destroy`, cierra la ventana.

## Observaciones

- Es una implementación **en memoria**: no hay persistencia.
- `messagebox.showwarning` se usa para reforzar la validación del input vacío.
- `curselection()` devuelve una tupla con índices seleccionados. Aquí se usa `sel[0]` para el primero.

## Posibles extensiones

- Permitir **edición** de elementos existentes (doble clic → editar).
- Añadir persistencia en archivo o base de datos.
- Usar `Listbox` con `selectmode=tk.MULTIPLE` para permitir eliminar varios ítems a la vez.
- Añadir barra de desplazamiento (`Scrollbar`) para listas largas.

---
