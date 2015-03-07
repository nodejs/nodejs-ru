# Assert

    Стабильность: 2 - Стабильный
    

Этот модуль используется для написания модульных тестов для приложений, подключается через `require('assert')`.

## assert.fail(actual, expected, message, operator)

Создает исключение, которое отображает фактическое (`actual`) и ожидаемое (`expected`) значения, разделенные предоставленным оператором.

## assert(value, message), assert.ok(value[, message])

Проверяет истинность значения, данная проверка аналогична `assert.equal(true, !!value, message);`

## assert.equal(actual, expected[, message])

Проверка на равенство с приведением типов значений. Аналогично сравнению через (`==`).

## assert.notEqual(actual, expected[, message])

Проверка на неравенство без приведения типов, аналогично сравнению через (`!=`).

## assert.deepEqual(actual, expected[, message])

Проверка на глубокое равенство. Примитивные значения сравниваются с помощью оператора сравнения - равно (`==`). Не учитывает прототипы объектов.

## assert.notDeepEqual(actual, expected[, message])

Проверка на любое глубокое неравенство, противоположно `assert.deepEqual`.

## assert.strictEqual(actual, expected[, message])

Проверка на строгое равенство. Значения сравниваются без неявного приведения типов, аналогом данного сравнения является использование оператора (`===`).

## assert.notStrictEqual(actual, expected[, message])

Проверка на строгое неравенство. Значения сравниваются без неявного приведения типов, аналогом данного сравнения является использование оператора (`!==`).

## assert.deepStrictEqual(actual, expected[, message])

Проверка на глубокое строгое равенство. Значения сравниваются без неявного приведения типов, аналогом данного сравнения является использование оператора (`===`).

## assert.notDeepStrictEqual(actual, expected[, message])

Проверка на строгое глубокое неравенство. Противоположно `assert.deepStrictEqual`.

## assert.throws(block&#91;, error&#93;&#91;, message&#93;)

Ожидает, что блок (`block`) вызовет ошибку. Ошибка (`error`) может быть конструктором, регулярным выражением (`RegExp`) или проверочной функцией.

Проверка instanceof с помощью конструктора:

    assert.throws(
      function() {
         throw new Error("Wrong value");
      },
      Error
    );
    

Проверка сообщения об ошибке, используя регулярное выражение:

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      /value/
    );
    

Пользовательская проверка ошибок:

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      function(err) {
        if ( (err instanceof Error) && /value/.test(err) ) {
          return true;
        }
    },
      "unexpected error"
    );
    

## assert.doesNotThrow(block[, message])

Ожидает, что блок (`block`) не вызовет ошибку, см. `assert.throws` для подробной информации.

## assert.ifError(value)

Проверяет, является ли value не ложным, создает исключение, если value равно true. Полезно при тестировании первого аргумента, `error` в обратных вызовах.