## ЧАСТЬ 1: АРХИТЕКТУРА

Компонент QuestionCard:
    Header (questionId, сложность, прогресс, таймер)
    QuestionStem (текст вопроса / формулы),
    AnswerOptions (radio buttons или checkboxes),
    ActionButtons (submit / previous question),
    Explanation (conditional),
    DemoOverlay (conditional, блюр),

1) selectedAnswer и isChecked хранится в QuestionCard
2) при смене questionId сбрасывается весь локальный стейт QuestionCard
3) при быстрых переключениях с вопроса на вопрос делаем оптимистичную загрузку, кнопки дизейблятся на время загрузки,
   используется requestId для проверки актуальности ответа


## ЧАСТЬ 2: ПСЕВДОКОД

```js
globalState: {
    questionId: string,
    questions: QuestionDTO[],
    progress: ProgressDTO,
    isDemo: boolean,
    timer: number,
}


const QuestionCard = () => {
    state: {
        selectedAnswer: string,
        isChecked: boolean,
        isLoading: boolean,
        isExpanded: boolean, //если вопрос слишком длинный и мы решили сворачивать и разворачивать его текст
        showExplanation: boolean,
        error: Error
    }

    response: {
        isCorrect: boolean,
        explanation?: string,

    }

    //При переходе на след/пред вопрос
    onQuestionChange (newId: string):
        resetState()
        fetchQuestion(newId)
            .then(data => setData(data))
            .catch(handleError)

    //При выборе ответа
    onAnswerSelect (answerId):
        if !isChecked:
            setSelectedAnswer(answerId)
            highlightAnswer(answerId)

    //АПИ проверки ответа
    onCheckAnswer():
        if !selectedAnswer || isChecked || isLoading: 
            return
        
        setIsLoading(true)

        checkAnswer(questionId, selectedAnswer)
            .then(res => {
                setIsChecked(true)
                setShowExplanation(true)
                updateScore(res.isCorrect)
            })
            .catch(error => setToast('Ошибка проверки'))
            .finally(() => setIsLoading(false))

    //Сбрасывание стейта
    resetState():
        setSelectionAnswer(null)
        setIsChecked(false)
        setShowExplanation(false)
        setIsLoading(false)

    render():
        checkButton:
            disabled: !selectedAnswer || isChecked || isLoading
        answerOptions:
            disabled: isChecked || isLoading
        nextButton: 
            disabled: isLoading

        if isChecked && hasExplanation:
            showExplanation = true
}
```

## ЧАСТЬ 3: EDGE-CASES, UI-ПОВЕДЕНИЕ

1) UI может показать сообщение "Объяснение отсутствует" или просто не показывать плашку explanation

2) На месте текста вопроса рендерим формулы через KaTeX

3) Если текст вопроса слишком длинный:
    - делаем возможность прокрутки через overflow-y: auto и определения максимальной высоты и/или возможность свернуть/развернуть текст вопроса
    - возможность разбить текст вопроса на непосредственно заголовок ("Ответьте на вопрос" и т.п.) и подзаголовок, в котором размер текста будет меньше, что улучшит читаемость.

4) Показывает сообщение/тост с ошибкой или (если есть такая возможность) показываем исходный код формулы в виде текста, а также кнопку "Повторить загрузку"

5) После нажатия кнопки проверки ответа варианты ответа дизейблятся, потому изменить решение нельзя.

6) В демо-режиме explanation скрыт или заблюрен. Есть вариант в зависимости от текущей роли пользователя (с подпиской, без, демо) присылать с бэкенда минимальную информацию, чтобы не было возможности убрать блюр через код элемента. 

Также для демо-режима видна плашка, что контент доступен только после подписки и кнопка "подписаться", при клике на которую появляется баннер с текущими уровнями подписки и ценами.