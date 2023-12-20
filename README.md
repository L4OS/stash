# stash
Selection of some interesting examples for the Everest processor

### Slagheap SoC

В этом репозитории вы найдёте библиотеки, тесты и примеры програм для компьютера "Террикон", также известного как Slagheap SoC. Изначально "компьютер" был реализовано на плате ["Марсоход"](https://marsohod.org/). Slagheap SOC был достаточно простым компьютером, он не задействовал микросхему DRAM и общался с внешним миром через UART порт. 
Главной особенностью этого компьютера была оригинальная система команд архитектуры CISC ([Complex Instruction Set Computer](https://habr.com/ru/companies/selectel/articles/542074/)).
Система команд Everest отличается высокой плотностью кода и оптимизирована для потокового исполнения.

<sup>Архитектура в шутку названная была названа "Эверест", как бы в противовес "Эльбрусам". </sup>

![Everest instruction codes map](https://everest.l4os.ru/wp-content/uploads/2015/02/MAP_EVER_1_1.png)

Система команд Эверест обладает высоким потенциалом расширения - на карте показаны незадействованные коды операций.

### Пример функции (подрограммы) на языке "макроассемблер Эверест"

Пример ассемблерного кода для "Террикона". Функция выводит строку в последовательный порт.
Использована простая запись без именования регистров и констант.
<code>
; --- Функция вывода строки
; Вход: R1 - адрес строки
; Выход: R0 - количество переданных символов

function	_puts
	push	r15
	push	r2
	mov	r0, r1		; Копирование указателя на строку
load_word:
	mov	r2, (r0)	; Загрузка машинного слова (4-символа)
	load	r4, 4		; Количество байт в машинном слове
check_byte:
	rol	r2, 8		; Циклический сдвиг на 8 бит влево
	load	r3, 0xff	; Загрузка восьмибитной константы
	and	r3, r2		; Проверка на конец строки
	je	done		; Вывод строки закончен
	call	_putchar        ; Вызов подпрограммы вывода символа
	inc	r0, 1		; Инкремент указателя
	dec	r4, 1		; Декремент счётчика
	jne	check_byte	; Следующий символ
	jmp	load_word	; Повтор операции
done:
	clc			; Сброс переноса
	subc	r0, r1		; Подсчёт количества выведенных символов
	pop	r2
	pop	r15
	return			; Возврат из функции
end
</code>

Это пример не самой простой функции. Сложность заключается в разрядности операций. Процессорное ядро Эверест и система команд на момент публикации не имеет инструкций для операций с 8-ми и 16-ти быитными регистрами, поэтому их отсутствие приходится компенсировать сдвигами и масками.

### Виртуальный компьютер "Террикон"

В процессе отладки SoC была реализована программная модель микропроцессора. В процессе тестирования программная модель приросла виртуальным видеоадаптером и клавиатурой в дополнение к виртуальному алфавитно-цифровому терминал. Технические характеристики виртуального компьютера "Террикон": 
- 32 бита;
- 16 регистров регистров общего назначения;
- 64 регистра  регистра сообщений;
- 32 килобайта оперативной памяти, может быть увеличена до 2 гигабайт;
- консоль-терминал, может быть использована для отладки;
- видеоадаптер 640х480 с цветностью 32 бита на пиксель;
- виртуальная клавиатура;
- счётчик тактов хоста;
- виртуальная CMOS память 16 регистров.

<!-- позиционно независимый код который не привязанн к конкретным адресам -->

### Application binary interface (ABI)

Базовые правила написания программ для архитектуры Эверест: 
1. Адрес возврата из функции (подпрограммы) помещается в регистр общего назначения R15. Если функция использует этот регистр или вызывает другие функции (подпрограммы) его необходимо где-нибудь сохранять, обвчно на стеке.
2. Младшие восемь регистров - R0-R7 могут быть использованы для передачи аргументов функциям (подпрограмам), они так же могут измениться в процессе работы функции. Регистр R0 используется для возвращения результата выполнения функции. Старшие восемь регистров R8-R15 подпрограмма не должна менять. Если она их использует, то должна сохранить при входе в функцbю и восстановить при выходе.
3. Регистр R4 используется в качестве указателя стека. Это соглашение. Аппаратный стек архитектура не поддерживает.
4. Система команд содержит префиксы. Префиксы могут менять логику работы команд. Следующая за префиксом инструкция сбрасывает префикс.


   
