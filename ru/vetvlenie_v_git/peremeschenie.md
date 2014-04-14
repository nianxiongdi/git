# Перемещение

В Git'е есть два способа включить изменения из одной ветки в другую: `merge` (слияние) и `rebase` (перемещение). В этом разделе вы узнаете, что такое перемещение, как его осуществлять, почему это удивительный инструмент и в каких случаях вам не следует его использовать.

## Основы перемещения

Если мы вернёмся назад к одному из ранних примеров из раздела про слияние (см. рис. 3-27), увидим, что мы разделили свою работу на два направления и сделали коммиты на двух разных ветках.


![](http://git-scm.com/figures/18333fig0327-tn.png)

Рисунок 3-27. Впервые разделенная история коммитов.

Наиболее простое решение для объединения веток, как мы уже выяснили, команда `merge`. Эта команда выполняет трёхходовое слияние между двумя последними снимками состояний из веток (C3 и C4) и последним общим предком этих двух веток (C2), создавая новый снимок состояния (и коммит), как показано на рисунке 3-28.


![](http://git-scm.com/figures/18333fig0328-tn.png)

Рисунок 3-28. Слияние ветки для объединения разделившейся истории разработки.

Однако, есть и другой путь: вы можете взять изменения, представленные в C3, и применить их поверх C4. В Git'е это называется _перемещение_ (rebasing). При помощи команды `rebase` вы можете взять все изменения, которые попали в коммиты на одной из веток, и повторить их на другой.

Для этого примера надо выполнить следующее:

	$ git checkout experiment
	$ git rebase master
	First, rewinding head to replay your work on top of it...
	Applying: added staged command

Перемещение работает следующим образом: находится общий предок для двух веток (на которой вы находитесь сейчас и на которую вы выполняете перемещение); для каждого из коммитов в текущей ветке берётся его дельта и сохраняется во временный файл; текущая ветка устанавливается на тот же коммит, что и ветка, на которую выполняется перемещение; и, наконец, одно за другим применяются все изменения. Рисунок 3-29 иллюстрирует этот процесс.


![](http://git-scm.com/figures/18333fig0329-tn.png)

Рисунок 3-29. Перемещение изменений, сделанных в C3, на C4.

На этом этапе можно переключиться на ветку `master` и выполнить слияние-перемотку (fast-forward merge) (см. рис. 3-30).


![](http://git-scm.com/figures/18333fig0330-tn.png)

Рисунок 3-30. Перемотка ветки master.

Теперь снимок состояния, на который указывает C3', точно такой же, как тот, на который указывал C5 в примере со слиянием. Нет никакой разницы в конечном результате объединения, но перемещение выполняется для того, чтобы история была более аккуратной. Если вы посмотрите лог для перемещённой ветки, то увидите, что он выглядит как линейная история работы: выходит, что вся работа выполнялась последовательно, когда в действительности она выполнялась параллельно.

Часто вы будете делать это, чтобы удостовериться, что ваши коммиты правильно применяются для удалённых веток — возможно для проекта, владельцем которого вы не являетесь, но в который вы хотите внести свой вклад. В этом случае вы будете выполнять работу в какой-нибудь ветке, а затем, когда будете готовы внести свои изменения в основной проект, выполните перемещение вашей работы на `origin/master`. Таким образом, владельцу проекта не придётся делать никаких действий по объединению — просто перемотка (fast-forward) или чистое применение патчей.

Заметьте, что снимок состояния, на который указывает последний коммит, который у вас получился, является ли этот коммит последним перемещённым коммитом (для случая выполнения перемещения) или итоговым коммитом слияния (для случая выполнения слияния), есть один и тот же снимок — разной будет только история. Перемещение применяет изменения из одной линии разработки в другую в том порядке, в котором они были представлены, тогда как слияние объединяет вместе конечные точки двух веток.

## Более интересные перемещения

Можно также сделать так, чтобы при перемещении воспроизведение коммитов начиналось не от той ветки, на которую делается перемещение. Возьмём, например, историю разработки как на рис. 3-31. Вы создали тематическую ветку (`server`), чтобы добавить в проект некоторый функционал для серверной части, и сделали коммит. Затем вы выполнили ответвление, чтобы сделать изменения для клиентской части, и несколько раз выполнили коммиты. Наконец, вы вернулись на ветку `server` и сделали ещё несколько коммитов.


![](http://git-scm.com/figures/18333fig0331-tn.png)

Рисунок 3-31. История разработки с тематической веткой, ответвлённой от другой тематической ветки.

Предположим, вы решили, что хотите внести свои изменения для клиентской части в основную линию разработки для релиза, но при этом хотите оставить в стороне изменения для серверной части, пока они не будут полностью протестированы. Вы можете взять изменения из ветки `client`, которых нет в `server` (C8 и C9), и применить их на ветке `master` при помощи опции `--onto` команды `git rebase`:

	$ git rebase --onto master server client

По сути, это указание “переключиться на ветку `client`, взять изменения от общего предка веток `client` и `server` и повторить их на `master`”. Это немного сложно; но результат, показанный на рисунке 3-32, довольно классный.


![](http://git-scm.com/figures/18333fig0332-tn.png)

Рисунок 3-32. Перемещение тематической ветки, ответвлённой от другой тематической ветки.

Теперь вы можете выполнить перемотку (fast-forward) для ветки `master` (см. рис. 3-33):

	$ git checkout master
	$ git merge client


![](http://git-scm.com/figures/18333fig0333-tn.png)

Рисунок 3-33. Перемотка ветки master для добавления изменений из ветки client.

Представим, что вы решили включить работу и из ветки `server` тоже. Вы можете выполнить перемещение ветки `server` на ветку `master` без предварительного переключения на эту ветку при помощи команды `git rebase [осн. ветка] [тем. ветка]` — которая устанавливает тематическую ветку (в данном случае `server`) как текущую и применяет её изменения на основной ветке (`master`):

	$ git rebase master server

Эта команда применит изменения из вашей работы над веткой `server` на вершину ветки `master`, как показано на рисунке 3-34.


![](http://git-scm.com/figures/18333fig0334-tn.png)

Рисунок 3-34. Перемещение ветки server на вершину ветки master.

Затем вы можете выполнить перемотку основной ветки (`master`):

	$ git checkout master
	$ git merge server

Вы можете удалить ветки `client` и `server`, так как вся работа из них включена в основную линию разработки и они вам больше не нужны. При этом полная история вашего рабочего процесса выглядит как на рисунке 3-35:

	$ git branch -d client
	$ git branch -d server


![](http://git-scm.com/figures/18333fig0335-tn.png)

Рисунок 3-35. Финальная история коммитов.

## Возможные риски перемещения

Всё бы хорошо, но кое-что омрачает всю прелесть использования перемещения. Это выражается одной строчкой:

**Не перемещайте коммиты, которые вы уже отправили в публичный репозиторий.**

Если вы будете следовать этому указанию, всё будет хорошо. Если нет — люди возненавидят вас, вас будут презирать ваши друзья и семья.

Когда вы что-то перемещаете, вы отменяете существующие коммиты и создаёте новые, которые похожи на старые, но являются другими. Если вы выкладываете (push) свои коммиты куда-нибудь, и другие забирают (pull) их себе и в дальнейшем основывают на них свою работу, а затем вы переделываете эти коммиты командой `git rebase` и выкладываете их снова, ваши коллеги будут вынуждены заново выполнять слияние для своих наработок. В итоге вы получите путаницу, когда в очередной раз попытаетесь включить их работу в свою.

Давайте рассмотрим пример того, как перемещение публично доступных наработок может вызвать проблемы. Представьте себе, что вы склонировали себе репозиторий с центрального сервера и поработали в нём. И ваша история коммитов выглядит как на рисунке 3-36.


![](http://git-scm.com/figures/18333fig0336-tn.png)

Рисунок 3-36. Клонирование репозитория и выполнение в нём какой-то работы.

Теперь кто-то ещё выполняет работу, причём работа включает в себя и слияние, и отправляет свои изменения на центральный сервер. Вы извлекаете их и сливаете новую удалённую ветку со своей работой. Тогда ваша история выглядит как на рисунке 3-37.


![](http://git-scm.com/figures/18333fig0337-tn.png)

Рисунок 3-37. Извлечение коммитов и слияние их со своей работой.

Далее, человек, выложивший коммит, содержащий слияние, решает вернуться и вместо слияния (merge) переместить (rebase) свою работу; он выполняет `git push --force`, чтобы переписать историю на сервере. Затем вы извлекаете изменения с этого сервера, включая и новые коммиты.


![](http://git-scm.com/figures/18333fig0338-tn.png)

Рисунок 3-38. Кто-то выложил перемещённые коммиты, отменяя коммиты, на которых вы основывали свою работу.

На этом этапе вы вынуждены объединить эту работу со своей снова, даже если вы уже сделали это ранее. Перемещение изменяет у этих коммитов SHA-1 хеши, так что для Git'а они выглядят как новые коммиты, тогда как на самом деле вы уже располагаете наработками из C4 в своей истории (см. рис. 3-39).


![](http://git-scm.com/figures/18333fig0339-tn.png)

Рисунок 3-39. Вы снова выполняете слияние для той же самой работы в новый коммит слияния.

Вы вынуждены объединить эту работу со своей на каком-либо этапе, чтобы иметь возможность продолжать работать с другими разработчиками в будущем. После того, как вы сделаете это, ваша история коммитов будет содержать оба коммита — C4 и C4', которые имеют разные SHA-1 хеши, но представляют собой одинаковые изменения и имеют одинаковые сообщения. Если вы выполните команду `git log`, когда ваша история выглядит таким образом, вы увидите два коммита, которые имеют одинакового автора и одни и те же сообщения. Это сбивает с толку. Более того, если вы отправите такую историю обратно на сервер, вы добавите все эти перемещенные коммиты в репозиторий центрального сервера, что может ещё больше запутать людей.

Если вы рассматриваете перемещение как возможность наведения порядка и работы с коммитами до того, как выложили их, и если вы перемещаете только коммиты, которые никогда не находились в публичном доступе — всё нормально. Если вы перемещаете коммиты, которые уже были представлены для общего доступа, и люди, возможно, основывали свою работу на этих коммитах, тогда вы можете получить наказание за разные неприятные проблемы.