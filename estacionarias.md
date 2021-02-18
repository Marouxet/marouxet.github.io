---
layout: default
---


# Ondas estacionarias |_Stationary waves_

Utilizando la libraría _pyqtgraph_, generamos una animación de una onda senoidal impactando contra una discontinuidad caracterizada por su impedancia de borde. En el [Repositorio](https://github.com/Marouxet/Acustica01) hay 4 archivos para 4 diferentes versiones de impedancia de borde:

* Impedancia Infinita 
* Impedancia cero
* Impedancia de material absorbente 

En este caso, también implementa un [objeto donde se calcula la impedancia del material absorbente](https://marouxet.github.io/matAbs), de acuerdo al modelo de Delany-Bazley 
* Impedancia de radiación

## Ejemplos de las simulaciones obtenidas

Impedancia infinita

![Impedancia Infinita](./assets/img/01Zinf.gif)

Impedancia Nula

![Impedancia Infinita](./assets/img/02Z0.gif)

Impedancia de un material absorbente

![Impedancia Infinita](./assets/img/03Zabs.gif)

Impedancia de radiación 

![Impedancia Infinita](./assets/img/04Zrad.gif)

## Sobre el código

En cada uno de los casos, se importan las librerías 

```python
from pyqtgraph.Qt import QtGui, QtCore 
import numpy as np
import pyqtgraph as pg
import sys
from fractions import Fraction as frac

```


Luego se define un objeto, llamado Plot2D, en cuyo método _init_ se configura la ventana de ploteo, los gráficos iniciales (llamados canvas), donde se cargan unas imágenes para ilustrar la condición de borde, se configuran ejes, unidades, etc.

```python
class Plot2D(object):
    
    def __init__(self):

        ## Imagen de fondo
        self.img = QtGui.QImage('002Zinf.png')
        try:
            # Convertir la imagen en el formato requerido por Qt
            self.img = self.img.convertToFormat(QtGui.QImage.Format_ARGB32_Premultiplied)
            self.imgArray = pg.imageToArray(self.img, copy=True)    
        except AttributeError:
                print ("Image not valid!")
                
        self.fondo1 = pg.ImageItem(image = self.imgArray)
        self.fondo2 = pg.ImageItem(image = self.imgArray)
        self.fondo3 = pg.ImageItem(image = self.imgArray)


        self.A = 1
        self.R = 1
        self.R_phase = 0
        self.f = 1     
        self.traces1 = dict()
        self.traces2 = dict()
        self.traces3 = dict() 
        self.phase = 0
        self.t = np.arange(-4*np.pi,0,0.01)
        self.app = QtGui.QApplication([])
        self.win = pg.GraphicsWindow(title = "Ondas Estacionarias - Impedancia infinita")
        
        self.win.setBackground('w')
        
  
        self.ejePresion = pg.AxisItem('left', showValues = False, text='Presion Acústica') 
        self.ejePresion2 = pg.AxisItem('left', showValues = False, text='Presion Acústica')
        self.ejePresion3 = pg.AxisItem('left', showValues = False, text='Presion Acústica')
       
        dictEjeX = {-3:'3\u03BB',-2.5:'2.5\u03BB',-2 :'2\u03BB',-1.5:'1.5\u03BB',-1:'\u03BB',-0.5:'0.5\u03BB'}

        self.ejeLambda = pg.AxisItem('bottom', showValues= True, text= 'Fracción de Longitud de onda')
        self.ejeLambda.setTicks([dictEjeX.items()])
        self.ejeLambda2 = pg.AxisItem('bottom', showValues= True, text= 'Fracción de Longitud de onda')
        self.ejeLambda2.setTicks([dictEjeX.items()])
        self.ejeLambda3 = pg.AxisItem('bottom', showValues= True, text= 'Fracción de Longitud de onda')
        self.ejeLambda3.setTicks([dictEjeX.items()])
       
        self.canvas = self.win.addPlot(axisItems = {'left': self.ejePresion , 'bottom':self.ejeLambda})
        self.canvas.addItem(self.fondo1) 
        self.canvas.showGrid(x = True, y = False, alpha = 0.7)    
        self.fondo1.setZValue(10)  # Imagen delante de otras
        self.fondo1.setRect(pg.QtCore.QRectF(0, -2.1, 1.75, 4.1)) # Dimensiones de imagen dentro del grafico (ajustado a mano)
        self.canvas.setYRange(-2, 2, padding=0)   
        self.canvas.setXRange(-3.1, 1.5, padding=0)  
       
        self.canvas.getAxis('left').setLabel('Presión ')
        
        
        self.win.nextRow()
        self.canvas2 = self.win.addPlot(axisItems = {'left': self.ejePresion2 , 'bottom':self.ejeLambda2})
        self.canvas2.showGrid(x = True, y = False, alpha = 0.7)      
        self.canvas2.addItem(self.fondo2)
        self.fondo2.setZValue(10)  # Imagen delante de otras
        self.fondo2.setRect(pg.QtCore.QRectF(0, -0.1, 1.75, 5.1)) # Dimensiones de imagen dentro del grafico (ajustado a mano)
        self.canvas2.setYRange(0, 5, padding=0)
        self.canvas2.setXRange(-3.1, 1.5, padding=0)  
        self.canvas2.addLegend()
        self.canvas2.getAxis('left').setLabel('Presión al cuadrado')
        
        self.win.nextRow()
        self.canvas3 = self.win.addPlot(axisItems = {'left': self.ejePresion3 , 'bottom':self.ejeLambda3})
        self.canvas3.showGrid(x = True, y = False, alpha = 0.7)    
        self.canvas3.addItem(self.fondo3)
        self.fondo3.setZValue(10)  # Imagen delante de otras
        self.fondo3.setRect(pg.QtCore.QRectF(0, -2.1, 1.75, 4.2)) # Dimensiones de imagen dentro del grafico (ajustado a mano)
        self.canvas3.setYRange(-2 , 2, padding=0)
        self.canvas3.setXRange(-3.1, 1.5, padding=0)  
        self.canvas3.getAxis('bottom').setLabel('Distancia a la discontinuidad')
        self.canvas3.getAxis('left').setLabel('Velocidad')

 ```
Los otros métodos del programa son:

_animation_ inicia el timer y lo configura para que cada un intervalo de 75 mS se ejecute el método _update_. Luego ejecuta el método _start_

 ```python 
 def animation(self):
        timer = QtCore.QTimer()
        timer.timeout.connect(self.update)
        timer.start(75)
        self.start()
 ```

_start_ inicia la aplicación (tomado de la documentación de pytqgraph)

 ```python 
    def start(self):
        if (sys.flags.interactive != 1) or not hasattr(QtCore, 'PYQT_VERSION'):
            QtGui.QApplication.instance().exec_()
 ```
 _update_ es el método que se ejecuta cada 75 ms. Define las señales seno y coseno. Cambia la fase en cada repetición y carga los valores en tres diccionarios (uno por gráfico), llamado _trace_, _trace2_, _trace3_. 

 ```python 
 def update(self):
        s = self.A*np.sin(2 * np.pi * self.f*(-self.t) + self.phase )
        c = self.A*self.R*np.sin(2 * np.pi * self.f* self.t + self.phase + self.R_phase)
        sume = s + c
        s2 = np.power(s, 2)
        c2 = np.power(c, 2)
        sume2 = np.power(sume, 2)

        
        vel_inc=self.A*np.cos(2 * np.pi * self.f*(-self.t+10) + self.phase)
        vel_ref=self.A*-self.R*np.cos(2 * np.pi * self.f* self.t + self.phase)
        vel_sum = vel_inc+vel_ref
        
        
        self.trace("inc", self.t, s)
        self.trace("ref", self.t, c)
        self.trace("sum", self.t, sume)
        self.trace2("inc", self.t, s2)
        self.trace2("ref", self.t, c2)
        self.trace2("sum", self.t, sume2)
        self.trace3("inc", self.t, vel_inc)
        self.trace3("ref", self.t, vel_ref)
        self.trace3("sum", self.t, vel_sum)
        self.phase += 0.1

 ```
_trace_ son tres métodos, que leen la información del diccionario y la cargan en los ejes, actualizando la info cada 75 ms y generando la sensación de movilidad.

 ```python
    def trace(self,name,dataset_x,dataset_y):
        if name in self.traces1:
            self.traces1[name].setData(dataset_x,dataset_y)
        elif name == "inc":         
            self.traces1[name] = self.canvas.plot(pen=pg.mkPen('r', width=2, style=QtCore.Qt.DashLine), name="Incidente")
        elif name == "ref":
            self.traces1[name] = self.canvas.plot(pen=pg.mkPen('g', width=2, style=QtCore.Qt.DashLine),name="Reflejada")
        elif name == "sum":
            self.traces1[name] = self.canvas.plot(pen=pg.mkPen('k', width=2.5),name="Suma")
    
    def trace2(self,name,dataset_x,dataset_y):
        if name in self.traces2:
            self.traces2[name].setData(dataset_x,dataset_y)
        elif name == "inc":         
            self.traces2[name] = self.canvas2.plot(pen=pg.mkPen('r', width=2, style=QtCore.Qt.DashLine), name = "Incidente" )
        elif name == "ref":
            self.traces2[name] = self.canvas2.plot(pen=pg.mkPen('g', width=2, style=QtCore.Qt.DashLine), name = "Reflejada")
        elif name == "sum":
            self.traces2[name] = self.canvas2.plot(pen=pg.mkPen('k', width=2.5), name = "Suma")
            
    def trace3(self,name,dataset_x,dataset_y):
        if name in self.traces3:
            self.traces3[name].setData(dataset_x,dataset_y)
        elif name == "inc":         
            self.traces3[name] = self.canvas3.plot(pen=pg.mkPen('r', width=2, style=QtCore.Qt.DashLine) )
        elif name == "ref":
            self.traces3[name] = self.canvas3.plot(pen=pg.mkPen('g', width=2, style=QtCore.Qt.DashLine))
        elif name == "sum":
            self.traces3[name] = self.canvas3.plot(pen=pg.mkPen('k', width=2.5))         
 ```
Por último, se instancia un objeto _Plot2D()_ y se llama a animation para comenzar

 ```python
p = Plot2D()
p.animation()


```

[back](./)
