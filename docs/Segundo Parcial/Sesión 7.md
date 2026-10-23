# Sesión numero 7 de Mecatrónica (2 de octubre de 2026)
La clase de hoy, consisitio en seguir viendo temas de clase. Enfocado nuevamente en el proyecto a realizar. Aunque nosotros ya estemos realizando nuestro vehiculo, Oliver nos empezó a enseñar por asi decirlo como es que se va automatizar por medio de camaras, osea, visión por computadora. 

### Camara

Para comenzar, se nos brindo este codigo

```
import cv2

cap = cv2.VideoCapture(0)  # prueba 0/1/2 si tienes varias cámaras
if not cap.isOpened():
    raise RuntimeError("No se pudo abrir la cámara")

# Opcional: fija resolución (si la cámara lo permite)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

while True:
    ok, frame = cap.read()
    if not ok:
        break

    cv2.imshow("Frame", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):  # presiona 'q' para salir
        break

cap.release()
cv2.destroyAllWindows()
```
Despúes, nos explico las disitintas paletas de colores que hay, RGB, HSL, HSV, y del como cambiar a cierto color y como elegir en que región colocar un color y así.

```
import numpy as np
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

# ejemplo: detectar verdes (ajusta a tu iluminación)
low  = np.array([35,  60,  60], dtype=np.uint8)
high = np.array([85, 255, 255], dtype=np.uint8)
mask = cv2.inRange(hsv, low, high)

seg = cv2.bitwise_and(frame, frame, mask=mask)
cv2.imshow("Mask", mask)
cv2.imshow("Segmentado", seg)
```
Algo asi sería el fragmento de código.

### Detección de color

Aprovechando la habilidad de la cámara para dectectar colores, puedes hacer que la camara "detecte" en que partes de la camára, soea imagen, se esta proyectando dicho color.

Así termino quedando mi código masomenos:

```
import cv2
import numpy as np

cap = cv2.VideoCapture(0)  # prueba 0/1/2 si tienes varias cámaras
if not cap.isOpened():
    raise RuntimeError("No se pudo abrir la cámara")

# Opcional: fija resolución (si la cámara lo permite)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

while True:
    ok, frame = cap.read()
    if not ok:
        break

    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    
  
    low = np.array([35, 60, 60], dtype=np.uint8)
    high = np.array([85, 255, 255], dtype=np.uint8)
    ja un círculo verde en el centro del frame
    filtro = cv2.inRange(hsv, low, high)
    
    
    erosion = cv2.dilate(filtro, kernel, iterations=2)
    erosion = cv2.erode(erosion, kernel, iterations=2)
 
    filtrada = cv2.bitwise_and(frame, frame, mask=erosion)


    
    cv2.imshow("mask", filtro)
   cv2.imshow("Erosion", erosion)
    cv2.imshow("HSV", filtrada)

    # Presiona 'q' para salir del bucle
    if cv2.waitKey(1) & 0xFF == ord('q'):  
        break

cap.release()
cv2.destroyAllWindows()
```
(imagen andres y tu color azul)
Mi compañero y yo decidimos detectar el color azul.

Por último, olvier nos enseño a que la camara rodee ciertos objetos, cosa que no termine realizando en mi laptop, sin emmbargo, nos enseño masomenos un resultado final desde cu computadora. Al igual que un ejemplo de código en su página web.
```

# 1) Asegúrate de tener una máscara binaria limpia
mask = opening  # por ejemplo, tras apertura

# 2) Buscar contornos (usa RETR_EXTERNAL para externos)
contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

canvas = frame.copy()
for c in contours:
    area = cv2.contourArea(c)
    if area < 200:  # filtra ruido muy pequeño
        continue

    # Dibujo del contorno:
    cv2.drawContours(canvas, [c], -1, (0,255,0), 2)

    # Rectángulo axis-aligned:
    x, y, w, h = cv2.boundingRect(c)
    cv2.rectangle(canvas, (x,y), (x+w, y+h), (255,0,0), 2)

    # Círculo mínimo envolvente:
    (cx, cy), rad = cv2.minEnclosingCircle(c)
    cv2.circle(canvas, (int(cx), int(cy)), int(rad), (0,255,255), 2)

    # Centroides con momentos:
    M = cv2.moments(c)
    if M["m00"] != 0:
        cx_m = int(M["m10"] / M["m00"])
        cy_m = int(M["m01"] / M["m00"])
        cv2.circle(canvas, (cx_m, cy_m), 4, (0,0,255), -1)
        cv2.putText(canvas, f"({cx_m},{cy_m})", (cx_m+6, cy_m-6),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0,0,255), 1)

# canvas ahora contiene contornos y centros dibujados
```

Esa fue la clase del día de hoy. Algo confusa la verdad, diría que me atrase bastante itentando replciar el codigo, pero se logró