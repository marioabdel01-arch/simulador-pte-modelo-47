# Simulador PTE modelo 47: reproducción de Amin, Masada y Womack (1984)

Simulador en línea del controlador auto-sintonizable (STC) multivariable de
Amin, Masada y Womack (1984), aplicado a la caldera de la unidad termoeléctrica
de 600 MW modelada por Usoro (1977). Corre por completo en el navegador: no
necesita MATLAB ni instalar nada.

**Abrir el simulador:** https://marioabdel01-arch.github.io/simulador-pte-modelo-47/

**Guía en video** (unos 10 minutos, narrada y con subtítulos): botón «▶ Guía en video» de la
página, o https://marioabdel01-arch.github.io/simulador-pte-modelo-47/#video

**Modelo de 27 estados:** hay un simulador con la misma interfaz para el modelo estándar de
27 estados de Usoro (1977), con sus lazos PI originales: https://marioabdel01-arch.github.io/simulador-pte-modelo-27/

## Qué simula

- **Planta:** el modelo no lineal de 47 estados de Usoro, con sus lazos PI
  originales, integrado con Runge–Kutta de cuarto orden y paso de 0,1 s.
- **Variables:** se regulan la temperatura del vapor sobrecalentado (TSSO), la
  del recalentado (TRHO) y la presión del vapor (PSSO), manipulando la demanda
  maestra de caldera (CBMD), la inclinación de quemadores (CXGG) y la
  atemperación del sobrecalentador (CWSY).
- **Controladores:**
  - el PI de Usoro;
  - el STC de Amin: ARX 3×3, N = 5, k = 1, T = 20 s, RLS con factor de
    olvido β, y ley de varianza mínima generalizada con ponderación λ;
  - el STC modificado: tabla de parámetros cada 5 % de carga y estimador
    congelado durante el transitorio. En la prueba de demanda real la tabla se
    interpola según la carga (parámetros y punto de operación) y el estimador
    solo se congela con cambios de carga mayores de 1 %/min (y 130 s después).
    En las pruebas de Usoro usa una tabla de 50 a 100 % cada 5 % (más 77,5 %),
    hecha con la planta original por el mismo procedimiento.
- **Pruebas:**
  - las cuatro rampas de carga de Usoro (1977, pruebas V.1 a V.4): de 100 a
    77,5 %, de 77,5 a 50 %, de 50 a 77,5 % y de 77,5 a 100 % a 15 %/min, con su
    estado inicial y la planta completa (con la atemperación del recalentador).
    Corren el PI, el STC y el STC modificado. Estas pruebas no son del artículo
    de Amin: con λ = 0,5 y β = 0,99, ni el STC ni el modificado mejoran al PI
    (media de IAE/PI de 4,42 y 4,70);
  - las dos rampas de Amin et al. (1984): de 90 a 70 % y de 70 a 90 % a
    10 %/min, donde se comparan el PI, el STC y el STC modificado;
  - **demanda real del SIN:** 24 h en tiempo real con la demanda horaria del
    Sistema Interconectado Nacional de un día real (CENACE, «Estimación de la
    Demanda Real del Sistema por Balance»), interpolada cada minuto y escalada
    para que el mínimo del día corresponda al 70 % y el máximo al 90 %, desde la
    hora de menor demanda y el estado estable al 70 %. Días: jueves 20/08/2026
    (pico del año), miércoles 11/03/2026 y domingo 18/01/2026 (la carga no baja
    del 70 %, mínimo técnico). Duración de 24, 12 o 6 h; registro cada 5 s. A
    velocidad máxima, una corrida de 24 h tarda alrededor de un minuto (dos con
    el PI de fondo).
- **Opciones:** λ, β y la duración.
- **Exportar:** al pausar o terminar una corrida se descargan un informe en PDF
  (configuración, las seis gráficas e índices), los datos en CSV
  (cada 1 s, o 5 s en la prueba de 24 h: demanda, salidas, consignas y mandos aplicados; la configuración y
  los índices van en el encabezado, en líneas que empiezan con `#`) y las seis
  gráficas en PNG. En MATLAB:
  `T = readtable('archivo.csv', 'CommentStyle', '#');`
- **Pestaña «Planta»:** el esquema de la unidad con los valores del modelo en
  el instante actual y un panel de gráficas por área, que se abre al pulsar el
  equipo en el esquema o su botón. Los paneles muestran las 24 variables que
  registra y grafica Usoro (1977), repartidas entre caldera, hogar y
  ventiladores, sobrecalentador, recalentador, turbina, condensador y agua de
  alimentación.

## Verificación

El código es una traducción a JavaScript del simulador en MATLAB de la
reproducción. Las seis corridas del artículo de Amin (PI, STC y STC modificado
en las dos rampas) coinciden con las de MATLAB con diferencias menores a 10⁻⁶
en las salidas, y los índices IAE son idénticos. En las cuatro pruebas de
Usoro, las 24 variables que él registra coinciden con MATLAB
(`pte47_run(N, 'clampStates', true)`) con un error relativo menor a 10⁻¹⁰.
En la prueba con demanda real (perfil del 20/08/2026, primeras 2 h), el PI, el
STC y el STC modificado coinciden con el simulador en MATLAB (`amin_demanda` y
`amin_lazo`) con diferencias menores a 10⁻¹⁰ en las salidas. En las cuatro
pruebas de Usoro, el PI, el STC y el STC modificado coinciden con MATLAB
(`amin_run_usoro`) con una diferencia relativa de IAE menor a 3·10⁻⁸
(desde la carpeta `fuentes`: `node verificacion/verifica_usoro.js`).
Para comprobarlo:

```
node verificacion/verifica.js
```

## Estructura

| Archivo | Contenido |
|---|---|
| `index.html`, `css/`, `js/app.js` | interfaz |
| `guia_simulador.mp4` | guía en video del simulador |
| `js/digpte47.js`, `js/funciones.js` | modelo de 47 estados de Usoro (traducido de MATLAB) |
| `js/sim.js` | RLS, ley de control, tabla por carga, perfiles de carga y lazo de simulación |
| `js/datos.js` | parámetros de la planta, modelo ARX, tabla por carga, covarianza inicial, estados estacionarios y perfiles de demanda del SIN |
| `img/esquema_planta.svg` | esquema de la planta (vectorial) |
| `verificacion/` | corridas de referencia de MATLAB y el script que las compara |

## Insertarlo en otra página

```html
<iframe src="https://marioabdel01-arch.github.io/simulador-pte-modelo-47/"
        width="100%" height="1100" style="border:0"></iframe>
```

## Licencia

[MIT](LICENSE): se puede usar, copiar y modificar libremente, siempre que se
conserve el aviso de autoría. Si lo usa en un trabajo académico, cite la
reproducción.

## Créditos

- Usoro, P. B. (1977). *Modeling and simulation of a drum boiler-turbine power
  plant under emergency state control* [Tesis de maestría, MIT].
- Quintero-Mármol Márquez, E. (2025). *Plantas termoeléctricas: análisis
  dinámico, modelado, simulación y control*. TecNM/CENIDET. (Esquema de la planta, redibujado en formato vectorial.)
- Quintero-Mármol Márquez, E. (2026). *Programación asistida por inteligencia
  artificial generativa de modelos de proceso para plantas termoeléctricas: el
  caso del modelo Usoro de 47.º orden* [Documento de trabajo]. TecNM/CENIDET.
  (Conversión del modelo de FORTRAN 77 a MATLAB.)
- Amin, M., Masada, G. Y., y Womack, B. F. (1984). Application of a
  multivariable self-tuning controller to a power plant boiler. *Proceedings of
  the 1984 American Control Conference*, 339–346.

Reproducción y simulador: Ing. Mario Abdel Guerra Pérez y Dr. Enrique
Quintero-Mármol Márquez, TecNM/CENIDET, Departamento de Ingeniería
Electrónica.

El PDF se genera con [jsPDF](https://github.com/parallax/jsPDF) 2.5.1 (licencia MIT),
incluido en `js/jspdf.umd.min.js`.
