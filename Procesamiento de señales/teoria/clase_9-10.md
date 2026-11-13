# Clase 9/10 — LoRa

## ¿Qué es LoRa?

**LoRa** (de *Long Range*) es una tecnología de comunicación por radio cuya modulación permite enviar pequeñas cantidades de datos con bajo consumo y buen alcance. Se utiliza, por ejemplo, en sensores de temperatura, medidores y monitoreo ambiental. Su diseño favorece las comunicaciones de baja tasa de datos; el alcance y la autonomía dependen de las condiciones del enlace y de cuánto transmite el dispositivo. [Semtech](https://www.semtech.com/uploads/technology/LoRa/lora-and-lorawan.pdf)

Desde el punto de vista de procesamiento de señales, interesa entender **cómo convierte un símbolo digital en una forma de onda y cómo el receptor reconoce ese símbolo**. Ese es el enfoque del [PDF de la materia, de Lorenzo Vangelista](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf), publicado en 2017.

## LoRa y LoRaWAN

| Concepto | Qué describe |
| --- | --- |
| **LoRa** | La capa física: la modulación de la señal de radio que transporta los datos. |
| **LoRaWAN** | El protocolo de red: organiza la comunicación de dispositivos, gateways y servidores, e incorpora mecanismos de seguridad y gestión. |

En una red LoRaWAN típica, un sensor transmite por radio a uno o varios **gateways**. Estos reenvían los mensajes mediante una conexión IP a un servidor de red. Por eso, usar una radio LoRa no implica por sí solo tener acceso a Internet: hace falta la infraestructura correspondiente. [LoRa Alliance](https://lora-alliance.org/about-lorawan-old/)

## La idea de un chirp

Un **chirp** es una señal cuya frecuencia instantánea cambia con el tiempo. En un chirp lineal ascendente (*upchirp*) la frecuencia aumenta; en uno descendente (*downchirp*) disminuye.

Como ejemplo matemático de un chirp lineal complejo, antes de considerar el retorno a los bordes de la banda:

$$
x(t)=A\exp\left(j2\pi\left[f_0t+\frac{\mu}{2}t^2\right]\right).
$$

Aquí $A$ es la amplitud, $f_0$ la frecuencia inicial y $\mu$ la pendiente del barrido, en Hz/s. La frecuencia instantánea se obtiene derivando la fase y dividiendo por $2\pi$:

$$
f_i(t)=f_0+\mu t.
$$

La fase es cuadrática y la frecuencia es lineal. Esta distinción ayuda a interpretar la expresión de la señal. LoRa utiliza una variante de **Chirp Spread Spectrum (CSS)**: durante un símbolo, la frecuencia barre la banda y retorna al extremo opuesto cuando alcanza un borde. [Maleki y colaboradores, sección II](https://arxiv.org/html/2310.10503v1#S2)

## ¿Dónde está la información?

El PDF propone describir la modulación como **Frequency Shift Chirp Modulation (FSCM)**. La información está en el desplazamiento de frecuencia del chirp que corresponde a cada símbolo. El receptor debe determinar qué desplazamiento se transmitió.

El **factor de ensanchamiento**, o *spreading factor* ($SF$), determina el tamaño del alfabeto:

$$
M=2^{SF},\qquad s\in\{0,1,\ldots,M-1\}.
$$

Por ejemplo, para $SF=7$ hay $128$ símbolos posibles. Cada índice permite representar un grupo de $7$ bits en el modelo de modulación. Esto no significa que cada símbolo de un paquete lleve siete bits útiles de la aplicación: también intervienen la codificación y la estructura del paquete.

El artículo trabaja con un intervalo de muestreo $T=1/B$, siendo $B$ el ancho de banda. Por lo tanto:

$$
T_s=MT=\frac{2^{SF}}{B},\qquad R_s=\frac{1}{T_s}=\frac{B}{2^{SF}}.
$$

$T_s$ es la duración de un símbolo y $R_s$ la tasa de símbolos. El muestreo a $B$ corresponde al modelo complejo en banda base del artículo; no es una regla general para muestrear directamente cualquier señal real de radiofrecuencia. [PDF de la materia, sección II](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf)

## Parámetros y ejemplo numérico

Los parámetros principales incluyen el ancho de banda ($B$ o $BW$), el factor de ensanchamiento ($SF$) y la tasa de codificación. A ancho de banda fijo, aumentar $SF$ alarga los símbolos y puede mejorar la sensibilidad del receptor, a costa de reducir la tasa y aumentar el tiempo de transmisión. La codificación agrega redundancia para corregir errores. [Semtech, capítulo 2](https://www.semtech.com/uploads/technology/LoRa/lora-and-lorawan.pdf)

Para comparar duraciones, tomemos **$B=125\,000$ Hz**:

| $SF$ | $M=2^{SF}$ | Duración $T_s$ | Tasa $R_s$ |
| --- | ---: | ---: | ---: |
| 7 | 128 | 1,024 ms | 976,56 símbolos/s |
| 10 | 1024 | 8,192 ms | 122,07 símbolos/s |
| 12 | 4096 | 32,768 ms | 30,52 símbolos/s |

Estos valores se calculan con las ecuaciones anteriores. Pasar de $SF=7$ a $SF=12$ multiplica la duración de cada símbolo por $32$. **Son duraciones de símbolos, no de paquetes completos.**

En el modelo sin redundancia, la tasa de bits asociada al alfabeto es:

$$
R_{b,\mathrm{sin\ codificar}}=SF\,R_s=SF\frac{B}{2^{SF}}.
$$

La tasa útil de una aplicación será menor al incluir redundancia, cabeceras y demás sobrecarga.

## ¿Cómo se recupera el símbolo? Dechirping y FFT

La idea central del receptor del PDF es transformar el chirp recibido en un tono y luego identificar su frecuencia.

Para verlo sin depender de una convención particular de fase, llamemos $c_0[k]$ al chirp base de módulo uno. En el modelo discreto ideal, un símbolo puede escribirse como:

$$
x_s[k]=\frac{1}{\sqrt{M}}c_0[k]e^{j2\pi sk/M},\qquad k=0,\ldots,M-1.
$$

El factor exponencial introduce el desplazamiento que identifica a $s$. Si no hay ruido y multiplicamos por el conjugado del chirp base:

$$
d[k]=x_s[k]c_0^*[k]=\frac{1}{\sqrt{M}}e^{j2\pi sk/M}.
$$

Como $c_0[k]c_0^*[k]=1$, desaparece el barrido y queda un tono complejo. Esta operación se llama **dechirping**.

Luego se calcula una DFT de $M$ puntos, implementada mediante una FFT:

$$
D[p]=\sum_{k=0}^{M-1}d[k]e^{-j2\pi pk/M},\qquad
\hat{s}=\underset{p}{\operatorname{arg\,max}}\;|D[p]|^2.
$$

En este modelo ideal, el pico está en el índice $p=s$. Con ruido, se busca el índice con mayor energía. En un receptor real también hacen falta tareas de sincronización y corrección de errores. [PDF de la materia, sección III](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf)

```text
Muestras recibidas → multiplicación por el chirp conjugado → FFT → índice del mayor pico
```

## Ortogonalidad y alcance del análisis del PDF

El artículo demuestra que las formas de onda de símbolos distintos, para un mismo $SF$ y bajo su modelo discreto sincronizado, tienen producto interno nulo:

$$
\langle x_i,x_q\rangle=0\quad\text{si }i\ne q.
$$

Esto permite distinguirlas mediante proyecciones. No debe confundirse con afirmar que transmisiones con distintos factores de ensanchamiento nunca se interfieren: la coexistencia entre factores diferentes requiere un análisis adicional. [PDF, sección II](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf); [Maleki y colaboradores, secciones II y VI](https://arxiv.org/html/2310.10503v1)

El PDF también compara FSCM con FSK de la misma cardinalidad. En sus simulaciones obtiene igual desempeño en el canal de ruido blanco gaussiano aditivo (**AWGN**) y mejor desempeño de FSCM en el canal selectivo en frecuencia considerado. Esta conclusión corresponde a las condiciones de esa comparación, con tasa de error sin codificar; no demuestra superioridad universal frente a toda modulación FSK. [PDF, secciones IV y V](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf)

## Relación con procesamiento de señales

Este tema conecta varios conceptos de la materia:

- **Fase y frecuencia instantánea:** la derivada de la fase explica el barrido del chirp.
- **Señales complejas y muestreo:** permiten representar y procesar la señal en banda base.
- **Producto interno y ortogonalidad:** permiten comparar la señal recibida con los símbolos posibles.
- **DFT y FFT:** convierten esas comparaciones en una implementación eficiente.
- **Ruido y canal:** determinan qué tan confiable es la decisión del receptor.

Para estudiar, conviene poder explicar la cadena: **bits → índice de símbolo → chirp desplazado → canal → dechirping → FFT → estimación del símbolo**.

## Fuentes

1. **Vangelista, L. (2017).** *Frequency Shift Chirp Modulation: The LoRa Modulation*. IEEE Signal Processing Letters, 24(12), 1818–1821. [PDF de la materia](../material_teorico/Frequency_Shift_Chirp_Modulation_The_LoRa_Modulation.pdf). DOI: [10.1109/LSP.2017.2762960](https://doi.org/10.1109/LSP.2017.2762960). Base del modelo discreto, la ortogonalidad, el receptor y la comparación con FSK.
2. **Semtech (2024).** [LoRa and LoRaWAN, AN1200.86](https://www.semtech.com/uploads/technology/LoRa/lora-and-lorawan.pdf). Contexto tecnológico, parámetros de radio y diferencia entre capa física y red.
3. **LoRa Alliance.** [What is LoRaWAN Specification](https://lora-alliance.org/about-lorawan-old/). Arquitectura de dispositivos, gateways y servidores.
4. **Maleki, A.; Nguyen, H. H.; Bedeer, E.; Barton, R. (2023).** [A Tutorial on Chirp Spread Spectrum for LoRaWAN: Basics and Key Advances](https://arxiv.org/html/2310.10503v1). Complemento sobre chirps, generación, detección e interferencia.

Fuentes externas consultadas el 28/10/2026. El desarrollo numérico y las explicaciones intermedias son elaboraciones didácticas a partir de los modelos citados.
