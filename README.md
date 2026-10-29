# Simulador GCP · Asignación de capital de trabajo en logística

Método de **Gradientes Conjugados Precondicionado (GCP)** aplicado a un problema de asignación de capital de trabajo entre cinco líneas operativas de una empresa de transporte, logística y almacenamiento.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Sin frameworks](https://img.shields.io/badge/frameworks-ninguno-success)

---

## Tabla de contenidos

1. [Descripción](#descripción)
2. [Caso financiero](#caso-financiero)
3. [Formulación matemática](#formulación-matemática)
4. [Funcionalidades](#funcionalidades)
5. [Tecnologías utilizadas](#tecnologías-utilizadas)
6. [Estructura del proyecto](#estructura-del-proyecto)
7. [Instalación](#instalación)
8. [Ejecución local](#ejecución-local)
9. [Despliegue público](#despliegue-público)
10. [Arquitectura del código](#arquitectura-del-código)
11. [Los tres escenarios](#los-tres-escenarios)
12. [Módulo QA](#módulo-qa)
13. [Referencias](#referencias)
14. [Autor](#autor)

---

## Descripción

Este proyecto resuelve un sistema lineal **A x = b** de **5 × 5** mediante el **Método de Gradientes Conjugados Precondicionado (GCP)**, descrito por Suñagua S. (2020), y lo compara con **Gradientes Conjugados simple (GC)**. La aplicación permite:

- Editar la matriz **A** y el vector **b** de forma interactiva.
- Elegir entre tres precondicionadores: **Jacobi**, **SSOR** o **IC0 con umbral**.
- Ejecutar GC simple y GCP en paralelo.
- Calcular la solución de referencia mediante **factorización de Cholesky**.
- Medir errores por variable: **Ea**, **Er**, **Ep**.
- Calcular números de condición **kappa(A)** y **kappa(M^-1 A)**.
- Visualizar la **curva de convergencia** en escala logarítmica.
- Comparar iteración por iteración los residuos de GC y GCP.
- Ejecutar un **módulo de pruebas de software (QA)** con cuatro casos.

Todo está contenido en **un único archivo `index.html` autocontenido**, sin frameworks y sin dependencias externas.

---

## Caso financiero

Una empresa de transporte, logística y almacenamiento opera cinco líneas de servicio simultáneas que **comparten activos** (flota, bodega, conductores, sistemas). Por eso, el capital de trabajo asignado a una línea no puede decidirse de forma aislada.

Las cinco variables del modelo son:

| Variable | Línea operativa | Unidad |
|----------|-----------------|--------|
| x1 | Transporte de carga pesada (largo recorrido) | miles USD |
| x2 | Distribución urbana / última milla | miles USD |
| x3 | Almacenaje seco y bodegaje | miles USD |
| x4 | Cadena de frío | miles USD |
| x5 | Logística inversa y devoluciones | miles USD |

Cada **x_i** representa el capital de trabajo asignado a la línea i. El vector **x** es el plan de asignación que se busca.

Los coeficientes de **A** representan:

- **a_ii** (diagonal): costo propio de financiar la línea i.
- **a_ij = a_ji** (fuera de la diagonal): tasa de acoplamiento operativo entre las líneas i y j.
- **b_i**: cobertura operativa objetivo de la línea i.

La simetría de **A** es un supuesto de modelado explícito: la tasa de reasignación de capacidad de i a j se asume igual a la tasa inversa.

---

## Formulación matemática

### Sistema lineal
A x = b, A en R^(5x5), x, b en R^5

### Propiedades de A

La matriz **A** es:

- Simétrica: A = A^T
- Definida positiva: x^T A x > 0 para todo x distinto de cero

Se verifica la definición positiva por tres vías equivalentes:

1. **Teorema de Gershgorin:** si A es estrictamente diagonal dominante con diagonal positiva, todos sus autovalores son positivos.
2. **Factorización de Cholesky:** si A admite A = L L^T con L real y diagonal positiva, entonces A es definida positiva.
3. **Cálculo explícito de autovalores:** si todos los lambda_i > 0, A es definida positiva.

### Solución de referencia

Como A es simétrica definida positiva (SPD), se usa Cholesky para calcular la solución de referencia:
A = L L^T
L y = b
L^T x_ref = y


### Algoritmos iterativos

- **GC simple:** Gradientes Conjugados sin precondicionar.
- **GCP:** Gradientes Conjugados Precondicionado, algoritmo Mz de Suñagua (2020).

### Precondicionadores soportados

| Precondicionador | Definición | Cuándo usarlo |
|------------------|------------|---------------|
| Jacobi | M = diag(A) | Mal escalamiento diagonal |
| SSOR | M = (D/w + L) D^-1 (D/w + L)^T, w en (0,2) | Acoplamiento fuerte fuera de la diagonal |
| IC0 con umbral | M = L L^T (Cholesky incompleta) | Acoplamiento fuerte con estructura dispersa |

### Criterio de parada
|| r_k ||_2 / || b ||_2 < 1e-6

donde **r_k = b - A x_k** es el residuo en la iteración k.

### Métricas de error

Por cada componente i:
Ea_i = | x_ref_i - x_GCP_i |
Er_i = Ea_i / | x_ref_i | (si x_ref_i != 0)
Ep_i = Er_i * 100 %

Métricas globales:
Residuo relativo = || b - A x_GCP ||_2 / || b ||_2
kappa(A) = || A ||_2 * || A^-1 ||_2
kappa(M^-1 A) = número de condición de la matriz precondicionada

---

## Funcionalidades

- Edición interactiva de **A** (5x5) y **b** (5).
- Selección de **x0**, tolerancia y N_max.
- Tres escenarios precargados: **Ideal**, **Bajo estrés**, **Mal condicionado**.
- Validación de **simetría** y **definición positiva** antes de ejecutar.
- **Solución de referencia** por Cholesky.
- Ejecución paralela de **GC simple** y **GCP**.
- Selección de precondicionador: **Jacobi**, **SSOR**, **IC0**.
- Tabla de **errores por variable** (Ea, Er, Ep).
- Cálculo de **kappa(A)** y **kappa(M^-1 A)**.
- **Curva de convergencia** en SVG (escala logarítmica).
- **Tabla detallada de iteraciones** (residuos GC vs GCP, iteración por iteración).
- **Tabla comparativa final** entre escenarios.
- **Módulo QA** con cuatro casos de prueba ejecutables.
- **Bitácora de pruebas** integrada.
- Diseño **responsive** y modo claro con alto contraste.

