<template lang="pug">
.engine(v-t9n.deep="'docs'")
  h1 Движок
  p Этот класс позволяет обрабатывать данные на основе потоков в узлах и передавать их из выходных данных во входные. Движок не зависит от других компонентов редактора. Все, что ему нужно - это идентификатор, воркеры Компонентов и JSON данные
  Code(source="engineNode")
  p Он может работать независимо от Редактора

  h2 Обработка
  p Обход узлов выполняется следующим образом:
  img(:src="processImg")
  p Рассмотрим ситуацию, когда во время работы в редакторе нужно сразу получать результаты изменений (это легко сделать с событием 'process')
  Code(source="engineProcess")
  p 
    | В общем случае при каждом изиенении в схеме (узлы, соединения, данные узла) необходимо выполнять обработку. 
    | В связи с тем, что воркеры могут быть асинхронными, метод 'process' также является асинхронным.
    | Так как действия, провоцирующие обработку, могут выполняться не дожидаясь завершения предыдущей обработки, нам необходим метод `abort`, который ожидает завершения предыдущей обработки и гарантирует целостность данных.
  p Обычно есть какой-то основной узел, с которого должна начинаться обработка или все данные идут к нему, тогда вы можете указать его:
  Code(source="processMethod")
  p 
    | Это может быть важно в ситуациях, когда узел запрашивает данные одновременно из нескольких узлов.
    | Например, если стартовым узлом будет Output material, тогда верхняя и нижняя цепочка узлов будут обработаны параллельно.
    | Если начать обработку с узла Lightness, тогда обработка нижних узлов будет начата только после получения результата в текущем.
  img(:src="startNodeImg")
  p Также вы можете передавать дополнительные аргументы внутрь воркеров
  Code(source="processArgs")
  p Каждому воркеру, обработанному этим процессом, будут переданы аргументы
  Code(source="worker")
  p Если во время обработки возникает ошибка (обнаружена рекурсия, неправильный startId, неверные данные), вы можете получить его сообщение и данные
  Code(source="onError")

  h2 Кроссплатформенный Движок
  p Внимание! Текущая версия движка на С++ не совместима с версией 1.0.0, но может быть обновлена при необходимости
  ul
    li
      a(href="https://github.com/retejs/cpp-engine") Cpp Engine 
</template>

<code name="engineNode">
var engine = new Rete.Engine('demo@0.1.0');

engine.register(myComponent);
await engine.process(data);
</code>

<code name="engineProcess">
editor.on('process', async () => {
    await engine.abort();
    await engine.process(editor.toJSON());
});
</code>

<code name="processMethod">
engine.process(data, node.id);
</code>

<code name="processArgs">
engine.process(data, null, arg1, arg2);
</code>

<code name="worker">
worker(node, inputs, outputs, arg1, arg2) {
  outputs['num'] = node.data.num;
}
</code>

<code name="onError">
engine.on('error', ({ message, data }) => { });
</code>

<script>
import processImg from '../assets/process.gif';
import startNodeImg from '../assets/processStartNode.png';

export default {
  data() {
    return {
      processImg,
      startNodeImg
    }
  }
}
</script>
