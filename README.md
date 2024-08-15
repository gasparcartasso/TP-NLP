# TP-NLP
La entrega consta de tres documentos importantes:
  - 'pruebas.ipynb' en donde dado en enunciado se hiceron pruebas con distintos modelos y parametros para llegar al mejor modelo posible y tambien es la notebook de entrenamiento
  - 'opeansearch_data_model.py' aqui es donde esta la estructura de los datos para la base de datos vectorial
  - 'daily predictions.ipynb' que toma el modelo entrenado y la base de datos creada para realizar la tarea diaria de agrupar los documentos y buscar lo que sea necesario en la base de datos vectorial

# Pruebas
Esta etapa, cada vez que se quiera re-entrenar el modelo para que haga de nuevo para que los topicos se actualicen, se tiene que repetir. La variable para elegir el modelo ganador es el silhouette_score llamada mas adelante metrica.
El primer paso es importar los documentos, para la entrada del modelo se utilizo los titulos concatenados con el cuerpo del articulo como un solo valor llamado 'title_text'
  - Primer modelo:
    Como inicio se hizo una prueba con BERTopic en leguaje espanol, como modelo base, nos dio una metrica de 0.198
  - Segundo modelo:
    En esta etapa opte por agregar un SentenceEncoder multilenguaje para aplicarle a los 'title_text', un CountVectorizer con los stopwords en espanol agregando la posibilidad de bigramas para la agrupacion de los topicos y un ClassTfidfTransformer para agrupar los topicos. La metrica mejora a 0.206
  - Tercer modelo:
    Ademas de lo que tenia el segundo modelo, le agregue, UMAP para la reduccion de la dimensionalidad, los hiperparametros elegidos corresponden a la mejor metrica obtenida luego de varias iteraciones. La metrica mejora a 0.212
  - Cuarto modelo:
    Se agruega un HDBSCAN para la agrupacion de los clusters y con una metrica de 0.216 es el modelo ganador

La etapa de Summarizing es la que corresponde al modelo que le da en nombre al topico. La eleccion fue algo mas subjetivo ya que lo que busque fue que se resumiera en pocas palabras el topico y le de un nombre. Probe con un Sumarizer basico pero el que mejor me dio resultado con lo qeu buscaba fue un modelo de transformer T5ForConditionalGeneration pre entrenado con 'google-t5/t5-large'

La siguiente etapa es la de guardar lo necesario en la base de datos vectorial para su posterior consulta en produccion.
Alternativamente a lo hecho en clase, opte por que las entidades y keywords sean guardados como listas para despues facilitar en produccion el matcheo entre el documento nuevo con sus keywords y entidades contra el topico asignado.
Tambien hice un dump en joblib del topic model ganador para usarlo en mis inferencias

# Predicciones
Cada vez que se reciban noticias nuevas, se cargan en memoria, se sube el topic model guardado en joblib y se hace una inferencia del topico usando como entrada el titulo concatenado con el texto. Esto nos devolvera el topico mas probable y su probabilidad, luego de ver algunas predicciones opte por que el minimo de probablilidad para que se le asigne un topico que no sea el 'basura' sea 0.2 por que se veian algunos articulos con baja probablidad que en realidad no pertenecian al topico guardado.
Otra cosa distinta a lo hablado en clase es que mi busqueda en la base de datos vectorial es por topico y no por embeddings. Me parecio una alternativa mas logica y directa de usar, ya que el modelo guardado tiene la capacidad de hacer una inferencia de que topico pertence y cuan probable es y no hacer esa misma inferencia atravez de una similitud coseno.
Una vez encontrado el topico en la base de datos, se devuelven las keywords y entidades presentes en tanto el articulo como en el topico guardado.
El ultimo paso es guardar en un archivo los outputs del modelo (id, keywords y entidades matcheadas)

   
