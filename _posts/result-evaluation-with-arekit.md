Once relation extraction model is ready-to-use, it is expected to be evaluated and compared with the other approaches. 
Given a couple datasets of a `test` and `etalon` types, it is possble to assess the result model using the AREkit presets.

In this posts we cover a possible evaluation formats for Sentiment Relations of the Relation Extraction subtask.

> NOTE: The present **AREkit-0.22.1** has a limitation onto 2 or 3 class problems for evaluation [#363](https://github.com/nicolay-r/AREkit/issues/363).
We looking forward to address on this issue in further. 

Конексты TEST (без меток)
Контексты ETALON (с разметкой меток)
Predict TEST (разметка меток)
Важный момент, что в TEST данных больше, так как мы решаем задачу извлечения (extraction), поэтому нам нужно еще выделить контексты!!!

И затем, как имея такую информацию можно оценить результат:

На уровне документа (тогда есть агрегация и прочее, можно на свою статью сослаться, взять код метода из нивц).
На уровне отедельных отношений в рамках текста.
