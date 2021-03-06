.. _section_python:

Using rasa NLU from python
==========================
Apart from running rasa NLU as a HTTP server you can use it directly in your python program.
Rasa NLU supports both Python 2 and 3.

Training Time
-------------
For creating your models, you can follow the same instructions as non-python users.
Or, you can train directly in python with a script like the following (using spacy):

.. testcode::

    from rasa_nlu.converters import load_data
    from rasa_nlu.config import RasaNLUConfig
    from rasa_nlu.model import Trainer

    training_data = load_data('data/examples/rasa/demo-rasa.json')
    trainer = Trainer(RasaNLUConfig("config_spacy.json"))
    trainer.train(training_data)
    model_directory = trainer.persist('./models/')  # Returns the directory the model is stored in

Prediction Time
---------------

You can call rasa NLU directly from your python script. To do so, you need to load the metadata of
your model and instantiate an interpreter. The ``metadata.json`` in your model dir contains the
necessary info to recover your model:

.. testcode::

    from rasa_nlu.model import Metadata, Interpreter

    metadata = Metadata.load(model_directory)   # where model_directory points to the folder the model is persisted in
    interpreter = Interpreter.load(metadata, RasaNLUConfig("config_spacy.json"))

You can then use the loaded interpreter to parse text:

.. testcode::

    interpreter.parse(u"The text I want to understand")

which returns the same ``dict`` as the HTTP api would (without emulation).

If multiple models are created, it is reasonable to share components between the different models. E.g.
the ``'nlp_spacy'`` component, which is used by every pipeline that wants to have access to the spacy word vectors,
can be cached to avoid storing the large word vectors more than once in main memory. To use the caching,
a ``ComponentBuilder`` should be passed when loading and training models, e.g.:

.. testcode::

    from rasa_nlu.model import Metadata, Interpreter
    from rasa_nlu.components import ComponentBuilder
    config = RasaNLUConfig("config_spacy.json")

    # For simplicity we will load the same model twice, usually you would want to use the metadata of
    # different models
    builder = ComponentBuilder(use_cache=True)      # will cache components between pipelines (where possible)
    metadata_model = Metadata.load(model_directory)

    interpreter = Interpreter.load(metadata_model, config, builder)
    # the clone will share resources with the first model, as long as the same builder is passed!
    interpreter_clone = Interpreter.load(metadata_model, config, builder)


