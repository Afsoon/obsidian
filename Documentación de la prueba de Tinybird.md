# Tinybird assignment
## Development
Para ejecutar el proyecto primero hay que instalar todas las dependencias:

```
npm install
```

Una vez instalado las dependencias, solo tenemos que ejecutar el proyecto con el siguiente comando:

```
npm run dev
```

Si queremos forzar que las peticiones a Tinybird son lentas, y ver que la página carga sin esperar a sus respuesta, vamos al fichero `route` que se encuentra en `/app/routes/_index/route.tsx`, y en la línea 38, cambiamos la variable `NO_DELAY` por `SEE_LOADING`.

En el caso de querer ver el mensaje de error, de la response no se ha resuelto, entonces en la misma línea, cambiar la variable por `SEE_ERROR`.
## Decisions
### Framework
El proyecto se ha realizado usando Remix, concretamente la plantilla Indie stack. ¿Por qué he decido usar Remix y una plantilla en vez de NextJS 13/14?.
Por los siguientes motivos:
- Conocimiento del framework: Hacerlo con NextJS hubiera supuesto un trabajo adiccional el tener que aprender las nuevas features, esto se traduciría en una peor implementación y más tiempo de desarrollo.
- Muchas de las cosas implementadas en Remix puede ser implementadas en NextJS (Cuando este hablando de ello pondré), ya que se apoyan directamente en React.
- El uso de la plantilla para tener un esqueleto inicial y solo añadir las pocas librerías que necesitaba para hacerlo.
### UI Components
He usado Tremor ya que nos da una batería de componentes preparados para realizar visualizaciones e inputs distintos.
### Tests
Para los tests he decidido usar Storybook, junto con Remix-testing, y Playwright, cada librería para unos tipos de test en concreto. Por extensión 

#### ¿Por qué no Vitest/Jest junto con JSDOM/Happy-DOM?.

JSDOM/Happy-DOM su intención es intentar simular lo mas posible un navegador usando solo JS, y es posible que un test pase pero en los browsers no funcione y viceversa. Este tipo de tests aun tiene cabida, por ejemplo tienes que controlar los timers, ya que Playwright no lo permite por ahora, aunque existe un workaround.
#### ¿Por qué Storybook Interactions en vez de Playwright Components?

A primera vista ambos tienen el mismo cometido pero me inclino más por Storybook por ahora por lo siguientes motivos:
- Nos permite generar una documentación de nuestros componentes y a la vez nuevos dev saber como se comporta los componentes en un browser.
- Playwright components es aún experimental, mientras que Interactions no lo es.
- Interactions se apoya por debajo en Playwright.

### ¿Por qué he añadido Storybook y Playwright casi al final?

Si se sigue el hilo de los commits, se puede ver que lo he añadido al final después de implementar, pero idealmente hubiera sido puesto desde el principio. Este cambio de orden es debido a no tener un diseño inicial pensado e ir iterando, en otras palabras, he decidido favorecer velocidad frente hacerlo bien.

Si hubiera pensado todo desde un principio, diseño de los distintos estados de carga, error y cargado, el layout de la app, entonces hubiera desarrollado al menos sus stories y el componente a la vez, y el test de playwright cuando aplicará.

### ¿Por qué Playwright en vez de Cypress o Puppeteer?

Por conocimiento previo de la librería, entre los tres me parece que tiene una configuración más sencilla, una API muy parecida a Testing Library y uniformidad ya que Storybook Interactions se basa en ello.

#### Configuración de Playwright

La configuración se sale un poco de lo común, ya que cada instancia de test levantará un Express, con el código final, e iniciar Mock Service Worker, en adelante MSW. Esta configuración tan fuera de lo estándar, en vez de tener un MSW y Express configurado para toda la suite, es por los siguientes motivos:

- Esta configuración me da la posibilidad de controlar los fetchs que se hacen en el servidor de Express, en este caso las peticiones a Tinybird. Esto control me da la posibilidad de generar response lentas, response erronéas y tener una batería de tests para evitar regresiones. 
- Reescribir la petición inicial de la página sería mucho más complejo, entre el SSR y el Streaming Mode.
- Mis tests son más cercanos a un flow completo de usuario ya que estoy testeando el código de cliente y servidor, sin apenas mocks.

Esta configuración debería de ser posible de traducir a NextJS. 

#### Efectos negativos de mis decisiones
Lo mencionado anterior son las cosas buenas que me dan estas herramientas, lo malo que los tests en este caso no son tan rápidos que usando JSDOM/Happy-DOM pero en mi opinión prefiero una batería de tests más confiables que velocidad.

### UI/UX

Para la UI/UX he decido implementarlo usando Streaming*. He decidido ir por esta ruta ya que en el código se hace lo siguiente:
- Peticiones a servicios de terceros, en este caso Tinybird. Nunca podemos confiar que la red siempre es rápida y/o que el servidor siempre es rápido.
- Relacionado con lo anterior, este diseño nos permite reducir el TTFB.
- Se hace transformaciones en código de una de las response de Tinybird, para acomodar los datos a Tremor y no hacerlo en cliente. No se puede asumir siempre que el cliente es un dispositivo rápido. 

#### Efectos negativos de mis decisiones

##### Remix Streaming es fácil cometer un fallo
Hay que tener cuidado en como se ordenas las promesas, todas las promesas que tienen que resueltas si o si desde el principio tiene que ir al principio del Loader, el resto tiene que ir al final. Ahora son pocas promesas, pero si tenemos mas peticiones a terceros y promesas dependientes nos obliga 

La implementación más parecida en NestJS sería con Parallel Routes, y en este caso el ciclo de promesas esta aislado por ruta, no tendríamos el problema que existe en Remix.

##### La petición para obtener las zonas de taxi.
Actualmente solo hago un await, si esta response es lenta perdemos las ventajas de tener un TTFB muy bajo. Para reducir la probabilidad de este problema se podría cachear pero esto solo debería de resolverse si vemos que la mediana, p95 y p99 es muy alto, ya que hacer una cache tiene sus problemas asociados.

##### Los componentes de loading
Para evitar tener mucho CLS, hay que tener asegurarse que los componentes de loading ocupen el mismo espacio. Algunas veces suele ser difícil, en este caso buscar el valor lo mas cercano posible pero una acumulación de componentes con distintas alturas y anchos puede producirlo, el workaround es acumulativo. 

## Next steps

Ahora pasamos todos los aspectos que se han dejado fuera, por tiempo no se han implementado, o como seguiría iterando. Los he divido en bloques para evitar cambios de contextos entre un punto y otro.
### UI/UX
Hay varios aspectos de la UX que se han omitido en favor de la velocidad, pero que en caso de ocurrir fallaría catastróficamente la app.
- La primera es la dependencia que tenemos a la petición para obtener todas las zonas de un taxi. Puede ocasionar dos problemas
- Un TTFB lento ya que el servidor no manda ninguna respuesta hasta que se resuelva la promesa del fetch.
- Que el endpoint nos devuelva un 500 y no podamos hidratar correctamente los datos para mostrarlos en el gráfico.
- Mejorar los mensajes de error basados de si hemos tenido un Timeout, un 500 o un 400. Ahora mismo tenemos un mensaje genérico que no indica al usuario de como puede continuar.
- Añadir Error Boundary para en caso de haber un error, no haga bubble hasta la raíz.
### Infra
- Configurar correctamente un CI/CD que compruebe estilo de código, typechecking y los tests de playwright y storybook.
- Elegir una plataforma para hacer deploy y que permita Streaming, en este caso puede ser Vercel o una imagen de docker de NodeJS con el server de Remix Express.
- Mover las API TOKEN a un secrets manager, como puede ser Vault, Infiscal o cualquier cosa parecida.
### Implementación
- Sugar syntax: Crear un Tagged String que se encarga de hacer toda la lógica del query builder para la request a Tinybird.
- Meter los selecto dentro de un formulario en vez de apoyarnos que el hook de React Router que actualiza los search paramos refresque la página. Como se ha dicho antes, en cierto aspectos se ha hecho incapie en ir rápido.
- Investigar si es posible hacer el resultado final de los datos usando solo SQL y evitar tener que procesar los datos con JS, la parte de hidratación y moldeado para Tremor. Aunque se hace en servidor, lo suyo sería que se hiciera en Tinybird.
- Hacer un wrapper de los componentes usados, es decir, en vez de importar directamente desde Tremor, crear los componentes por nuestra lado y reimportarlo. Esto con miras en el futuro de si queremos migrar el componente, solo tengamos que modificarlo en un punto y no en varios. Quedaría la siguiente estructura:
```
app
|
|
|----> components
|		|
|		|---> Text
|		|---> Card
|----> routes 

// The components will implemented the following way

import { Text as TremorText } from '@tremor/react'

export const Text = TremorText;

// Then in our files we do the following import
import { Text } from '@src/components/Text'
```
### Testing
- Añadir tests que falta de search params y error paths en Playwright.
- Mejorar las funciones de generar datos para los tests, algunos valores ahora mismo no se generan a partir de otros.