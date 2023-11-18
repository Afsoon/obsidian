# Tinybird assignment
## Development
Como ejecutar el proyecto:
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
Para los tests he decidido usar Storybook, junto con Remix-testing, y Playwright, cada librería para unos tipos de test en concreto.

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

- Esta configuración me da la posibilidad de controlar los fetchs que se hacen en el servidor de Express, en este caso las peticiones a Tinybird. Esto control me da la posibilidad de generar response lentas, response erroneas y tener una batería de tests para evitar regresiones. 
- Reescribir la petición inicial de la página sería mucho más complejo, entre el SSR y el Streaming Mode.
- Mis tests son más cercanos a un flow completo de usuario ya que estoy testeando el código de cliente y servidor, sin apenas mocks.

Esta configuración s
## Next steps