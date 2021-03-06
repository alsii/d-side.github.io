---
author: D-side
comments: true
date: 2013-07-14 15:42:33+00:00
layout: post
slug: gms-shaders-color
title: 'GameMaker Studio: шейдеры, цвета'
wordpress_id: 611
tags:
- GameMaker
- GLSL
---

На сегодня у нас кое-что более интересное, чего в стандартных примерах GMS нет. Будем писать реально полезные эффекты. Причём так, чтобы они выполнялись быстро и качественно. Дозировка - один шейдер на пост.

Начнём с простого - допустим, вам нужно сделать нечто чёрно-белым.

Тут нет большого количества подходов с тем, **чем** это сделать. Цвет каждой точки зависит только от её входного цвета, и ничего больше. Но возникают вопросы с тем, **как** определить нужный оттенок серого. Поясню, о чём речь.

Сейчас немалая часть прикладных применений использует модели представления цветов RGB и HSV.

RGB - модель достаточно простая. В ней каждый цвет может быть собран из трёх цветов, интенсивность которых регулируется. Цвет в модели RGB - это три значения интенсивности: красного, зелёного и синего цветов. Чаще используется диапазон 0-255 (и целые числа), в шейдерах мы пользуемся диапазоном 0-1 (и не только целыми). Если интенсивность по всем каналам одинакова - получается белый, чёрный или серый. Соответственно, на выходе мы должны получить одно некое число и записать его во все компоненты цвета-результата.

HSV (также пользуются HSL, она другая, но похожая) - более "реальная" модель, которую чаще используют художники из-за простоты обращения с цветами с её помощью. Тут три вполне понятных параметра цвета. Hue, оттенок, это, так сказать, "точка на радуге", сама сущность цвета, в физике это длина световой волны. Saturation, насыщенность, это то, насколько цвет далёк от серого (0) к чистому выбранному цвету (1 или 255). Value (или lightness) - яркость цвета.

Достаточно очевидный подход - попробовать взять последний параметр из цвета в цветовой модели HSV, и перевести его в соответствующий серый цвет. Мы могли бы перевести цвет в HSV, сбросить насыщенность на ноль, снова собрать в RGB и получить похожий эффект - но зачем, если мы знаем устройство серых цветов, и можем сразу получить требуемое? К тому же, как оказалось, это не лучший способ получать серые цвета.

Я сначала подумал взять просто среднее арифметическое от RGB, и это часто применяемый способ. Но [пошарился по википедии](http://en.wikipedia.org/wiki/HSL_and_HSV#Lightness) и наткнулся на килограмм различных характеристик цвета, из которых самым полезным показался [luma](http://en.wikipedia.org/wiki/Luma_(video)), получаемый вот так:

\\( L = 0.30 \cdot R + 0.59 \cdot G + 0.11 \cdot B \\)

У этой формулы есть логичное объяснение. Для начала, отметьте, что \\( 0.30 + 0.59 + 0.11 = 1 \\). То есть, процедура похожа на среднее арифметическое, но у трёх базовых цветов разная степень влияния на яркость, воспринимаемая глазом. Есть взять этот цвет, этот и этот, то можно отметить, что глазу "яркость" этих цветов кажется разной. Преобладание в яркости зелёного цвета очевидно, мне даже пришлось цвет шрифта на нём сменить, чтобы слово "этот" было видно. Это и выражено в цифрах, на которые компоненты умножаются.

 Всё, у нас всё есть для решения. Вот прямо сейчас возьму и напишу код шейдера. Шейдер, как обычно, чисто фрагментный - вершинный оставляем пропускающим.

{% highlight glsl %}
//Шейдер "Безысходность"
varying vec2 v_vTexcoord;
varying vec4 v_vColour;

void main()
{
    // Вытаскиваем цвет из текстуры
    vec4 col = texture2D( gm_BaseTexture, v_vTexcoord );
    // Вычисляем "люму", пока кто-нибудь мне не
    // скажет русское название этой величины
    float luma = col.r * 0.30 + col.g * 0.59 + col.b * 0.11;
    // Снова пользуемся особенностью GLSL на операциях с векторами:
    col.rgb = vec3(luma, luma, luma);
    // FINISH HIM
    gl_FragColor = v_vColour * col;
}
{% endhighlight %}
