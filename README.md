Репозиторий-справочник готовых вспомогательных функций, JavaScript-интерфейсов и CEF-событий для Pawn. Создан для того, чтобы быстро находить и копировать нужные вызовы без поиска по игровым модулям.

---

## 📌 Содержание
1. [Документы и Удостоверения](#1-документы-и-удостоверения)
2. [Худ и Спидометр](#2-худ-и-спидометр)
3. [Диалоги и Авторизация](#3-диалоги-и-авторизация)
4. [Админ-панель](#4-админ-панель)
5. [Государственные системы (Алкотестер, Радар, Тоник)](#5-государственные-системы)
6. [Бизнес-интерфейсы (Автосалон, АЗС, Банкомат, Выкуп)](#6-бизнес-интерфейсы)

---

## 1. Документы и Удостоверения

### Паспорт РФ
* **JS Функция:** `showPassport(surname, name, patronymic, sex, birth, skinId)`
* **Макет интерфейса:** [Посмотреть скриншот](https://ibb.co/dsL1mhpv)

<summary>📋 Пример реализации на сервере (Команда /pass)</summary>

```pawn
CMD:pass(playerid, params[])
{
    if(sscanf(params, "d", params[0])) return SCM(playerid, COLOR_PREF, ""CMD_PREF"{ffffff} /pass [ид игрока, которому вы хотите показать свой паспорт]");
    if(!IsPlayerConnected(params[0])) return SCM(playerid, COLOR_PREF, ""CMD_PREF"{ffffff} Игрока с таким ID нет на сервере.");
    if(GetPVarInt(params[0], "spec") == 1) return SCM(playerid, COLOR_PREF, ""CMD_PREF"{ffffff} Вы должны находиться возле игрока.");
    
    new Float:p1, Float:p2, Float:p3; 
    GetPlayerPos(params[0], p1, p2, p3);
    if(!IsPlayerInRangeOfPoint(playerid, 15.0, p1, p2, p3)) return SCM(playerid, COLOR_PREF, ""CMD_PREF"{ffffff} Вы должны находиться возле игрока.");
    
    SetPlayerChatBubble(playerid, "показал(а) свой паспорт", 0xAF5DD2FF, 20.0, 9000);
    
    // Проверка на наличие CEF у получателя
    if(GetPVarInt(params[0], "hudcef") == 0)
    {
        // Дефолтный диалог, если CEF не работает
        format:str_small("" #C_YELLOW "Фамилия: {ffffff}%s\n" #C_YELLOW "Имя: {ffffff}%s\n"#C_YELLOW "Отчество: {ffffff}%s\n" #C_YELLOW "Дата рождения: {ffffff}%s\n" #C_YELLOW "Место проживания: {ffffff}г. Арзамас.\nДействительно по 14.05.2026.", pInfo[playerid][pSurname], pInfo[playerid][pRusname], pInfo[playerid][pOtchestvo], pInfo[playerid][pBirth]);
        SPD(params[0], d_null, info, "" #C_YELLOW "Паспорт гражданина РФ", str_small, "Закрыть", "");
    }
    else
    {
        SCM(params[0], 0x3399FFFF, !"Чтобы закрыть интерфейс нажмите клавишу {FFFFFF}\"ALT\"{3399FF}");
        SCM(params[0], 0x3399FFFF, !"Если интерфейс не закрылся введите /close");
        
        new sex_text[16];
        sex_text = (pInfo[playerid][pSex] == 1) ? ("Мужской") : ("Женский");

        // Вызываем эвент в CEF получателя
        cef_emit_event(params[0], "interface:set:showPassport", 
            CEFSTR(pInfo[playerid][pSurname]), 
            CEFSTR(pInfo[playerid][pRusname]), 
            CEFSTR(pInfo[playerid][pOtchestvo]), 
            CEFSTR(sex_text), 
            CEFSTR(pInfo[playerid][pBirth]), 
            CEFINT(pInfo[playerid][pSkin]) // Отрисовка фото по ID скина
        );

        SetPVarInt(params[0], "udostover", 1);
    }
    return 1;
}

```

### Ведомственные удостоверения гос. структур

#### 🔹 Клиентские события (JS)

Данные эвенты должны быть зарегистрированы на стороне CEF клиента для приема аргументов от сервера:

```javascript
// Судебная система
uCef.addEvent('interface:set:showSUD', (...args) => { showSUD(...args); });
uCef.addEvent('interface:set:hideSUD', () => { hideSUD(); });

// Министерство Внутренних Дел
uCef.addEvent('interface:set:showMVD', (...args) => { showMVD(...args); });
uCef.addEvent('interface:set:hideMVD', () => { hideMVD(); });

```

#### 🔹 Прототипы функций JS (Шаблоны аргументов)

* **Правительство:** `showGov(skinId, 'номер', 'Фамилия', 'Имя', 'Отчество', 'Должность');` | [Скриншот](https://ibb.co/YCkC3tD)
* **Следственный комитет:** `showSK(skinId, 'Номер_Удост', 'Фамилия', 'Имя', 'Отчество', 'Должность');`
* **МЧС:** `showMChS(skinId, 'Номер', 'День', 'Месяц', 'Год_Рожд', 'Фамилия', 'Имя', 'Отчество', 'Звание', 'Должность', 'Д_Выдачи', 'М_Выдачи', 'Г_Окончания');`
* **Прокуратура:** `showPROKUROR(targetid, номер, 'Фамилия', 'Имя', 'Отчество', тип, 'Должность', 'Отдел');`
* **ФСБ:** `showFSB(skinId, 'Фамилия', 'Имя', 'Отчество', 'Должность', 'Отдел');`
* **Военная Полиция:** `showVP(skinId, номер1, номер2, день, месяц, год, 'Фамилия', 'Имя', 'Отчество', 'ранг1', 'ранг2');`
* **ЦОРДД:** `showCORDD(targetid, номер, 'Фамилия', 'Имя', 'Отчество', 'инспектор');`
* **Суд:** `showSUD(skinId, 'День', 'Месяц', 'Год', 'Номер', 'Фамилия', 'Имя', 'Отчество', 'Должность');`
* **Росгвардия (ФСВНГ):** `showVNG(skinId, номер1, номер2, день, месяц, год, 'Фамилия', 'Имя', 'Отчество', 'ранг1', 'ранг2');`
* **МВД:** `showMVD(skinId, 'Номер', 'МВД-Код', 'День', 'Месяц', 'Год', 'Фамилия', 'Имя', 'Отчество', 'Звание', 'Должность');`

#### 🔹 Варианты вызова из Pawn:

**Через системный эвент `cef_emit_event` (Рекомендуется):**

```pawn
new player_skin = GetPlayerSkin(playerid);
SCM(params[0], 0x3399FFFF, !"Чтобы закрыть интерфейс нажмите клавишу {FFFFFF}\"ALT\"{3399FF}");

cef_emit_event(params[0], "interface:set:showSUD", 
    CEFINT(player_skin), 
    CEFSTR("16"), CEFSTR("мая"), CEFSTR("2026"),
    CEFSTR("СУД-0004"),
    CEFSTR(pInfo[playerid][pSurname]), CEFSTR(pInfo[playerid][pRusname]), CEFSTR(pInfo[playerid][pOtchestvo]),
    CEFSTR("Мировой судья")
);
SetPVarInt(params[0], "udostover", 1);

```

**Через прямое исполнение JS-строки `cef_execute_js`:**

```pawn
new p_skin = GetPlayerSkin(playerid);
SCM(params[0], 0x3399FFFF, !"Чтобы закрыть интерфейс нажмите клавишу {FFFFFF}\"ALT\"{3399FF}");

static cef_buf[256];
format(cef_buf, sizeof(cef_buf), "showGov(%d, '%s', '%s', '%s', '%s', '%s');", 
    p_skin, "000152", pInfo[playerid][pSurname], pInfo[playerid][pRusname], pInfo[playerid][pOtchestvo], "Губернатор"
);
cef_execute_js(params[0], cef_buf);
SetPVarInt(params[0], "udostover", 1);

```

### Автомобильные и Медицинские документы

* **Мед-карта:** `showMED(skinId, 'Срок_До', 'Фамилия', 'Имя', 'Дата_Рожд', 'Адрес', 'Врач-Должность', 'Место_Работы');`
* **ОСАГО:** `showPolis('ФИО_Клиента', 'Адрес', СерияНомер, '-', 0, 'Марка_Модель', 'VIN', 'Гос_Номер', Телефон, 'С_Даты', 'По_Дату', НомерДог);`
* **Водительское удостоверение:** `showVU('Фамилия', 'Имя Отчество', 'Дата_Рожд', кат_A, кат_B, кат_C, кат_D, 'Особые_Отметки', Номер_ВУ);` | [Скриншот](https://ibb.co/qGGpbZs)

#### Свидетельство о регистрации ТС (СТС)

* **Макет интерфейса:** [Посмотреть скриншот](https://ibb.co/0pq5wN9V)

```pawn
if(GetPVarInt(params[0], "hudcef") == 0)
{
    SCM(playerid, -1, "У игрока нет CEF лаунчера.");
}
else
{
    SCM(params[0], 0x3399FFFF, !"Чтобы закрыть интерфейс нажмите клавишу {FFFFFF}\"ALT\"{3399FF}");
    
    static string[512];
    format(string, sizeof(string), 
        "showSTS('%s', '%s', '%s', '%s', '%s', %d, '%s', %d, %d, '%s', '%s', %d, %d, '%s', '%s', '%s', '%s', '%s', '%s', %d);",
        "А777МР152", "XTA21070011223344", "ВАЗ 2107", "ЛЕГКОВОЙ", "B", 2026, "255, 0, 0", 74, 1500, "52 11", "654321", 1050, 1450, "9911223344", "Зенетти", "Николай", "Александрович", "Арзамас Ленина 12", "16.05.2026", 0
    );

    cef_execute_js(params[0], string);
    SetPVarInt(params[0], "udostover", 1);
}

```

---

## 2. Худ и Спидометр

### Функции управления худом (JS)

```javascript
showInterface('newHud-0', true); // Включить худ. Варианты ID: newHud-0, newHud-1, newHud-2, newHud-3
setBar('health', 85);           // Индикатор здоровья (0 - 100)
setBar('armour', 50);           // Индикатор брони (0 - 100)
setNeed('food', 180);           // Шкала голода (0 - 200)
setNeed('drink', 120);          // Шкала жажды (0 - 200)

// Спидометр бензинового ТС (Тип 0) + установка скорости
showInterface('car', true, 0);
document.getElementById('text_speed').innerText = '120';

```

* **Варианты внешнего вида худа:** [`newHud-0`](https://ibb.co/RkP7Hrny) | [`newHud-1`](https://ibb.co/vxh0Npwk) | [`newHud-2`](https://ibb.co/k6DGmbZp) | [`newHud-3`](https://ibb.co/zhB2wK10)

### Пакетная синхронизация из Pawn

Отправка всех динамических характеристик игрока в один поток:

```pawn
// Принудительно инициализируем оболочку худа на клиенте
cef_execute_js(playerid, "showInterface('newHud-0', true);");

new Float:hp, Float:armour;
GetPlayerHealth(playerid, hp);
GetPlayerArmour(playerid, armour);

new max_hp = 100;
new breath = 100;
new wanted = GetPlayerWantedLevel(playerid);
new weapon = GetPlayerWeapon(playerid);
new ammo = GetPlayerAmmo(playerid);
new max_ammo = 500; 
new money = GetPlayerMoney(playerid);

new Float:speed = 0.0;
if(IsPlayerInAnyVehicle(playerid))
{
    new Float:vx, Float:vy, Float:vz;
    GetVehicleVelocity(GetPlayerVehicleID(playerid), vx, vy, vz);
    speed = floatsqroot(vx*vx + vy*vy + vz*vz) * 179.28625;
}

// Отправляем системное обновление в пакетном виде
cef_emit_event(playerid, "game:data:playerStats", 
    CEFFLOAT(hp), CEFINT(max_hp), 
    CEFFLOAT(armour), CEFINT(breath), 
    CEFINT(wanted), CEFINT(weapon), 
    CEFINT(ammo), CEFINT(max_ammo), 
    CEFINT(money), CEFFLOAT(speed)
);

```

---

## 3. Диалоги и Авторизация

### Кастомные CEF Диалоги

```javascript
showDialog(id, 'Заголовок', '{FF0000}Первая строка текста\n{FFFFFF}Вторая строка текста', 'Кнопка_1', 'Кнопка_2') 

```

* **Примеры стилей:** [Стиль 1](https://ibb.co/MDR97NBW) | [Стиль 2](https://ibb.co/VYW5Kx9p)

### Окна авторизации и выбора спавна

```javascript
showLogin(2, "Zenettany", 1); // Окно авторизации С полем ввода защитного кода
showLogin(2, "Zenettany", 0); // Окно авторизации БЕЗ защитного кода
showLogin(0, "Nikolai_Zenetti"); // Форма регистрации аккаунта
showLogin(3, 1, 1);           // Интерактивное меню выбора точки спавна

```

### Меню персонажа (Статистика / Сводка)

```javascript
showMenu(
    "Zenettany", // Никнейм
    2,           // Статус VIP (2 = VIP+)
    15,          // Уровень (LVL)
    48,          // Очки опыта (EXP)
    1500,        // Донат валюта
    5450000,     // Наличные деньги
    12800000,    // Деньги на банковском счету
    1,           // Пол (1 = Мужской, 2 = Женский)
    0,           // Текущие варны (Предупреждения 0/3)
    "888888",    // Номер телефона
    1250,        // Баланс мобильного телефона
    5,           // Количество лет проживания в штате
    "ППС",       // Наименование текущей фракции/организации
    "[ОБ ДПС]",  // Название подразделения / отдела внутри фракции
    "Старший лейтенант", // Текущий ранг
    "Дальнобойщик",      // Подработка / Работа
    80,          // Процент голода
    95,          // Процент жажды
    "-1",        // Наличие черного списка (-1 = Отсутствует)
    "Zenetti", "Nikolai", "Alexandrovich", // Фамилия, Имя, Отчество
    "15.05.2009", // Дата рождения игрока
    1,           // ID используемого худа
    3            // Шаг текущего квестового прогресса
);

```

---

## 4. Админ-панель

### Клиентские события (JS)

```javascript
// Эвенты управления основной панелью администрирования
uCef.addEvent('interface:set:showAdminpanel', (...args) => { showAdminpanel(...args); });
uCef.addEvent('interface:set:hideAdminpanel', () => { hideAdminpanel(); });

// Эвенты управления панелью информации о транспорте при слежке (Recon Car)
uCef.addEvent('interface:set:showAdminpanelCar', (...args) => { showAdminpanelCar(...args); });
uCef.addEvent('interface:set:hideAdminpanelCar', () => { hideAdminpanelCar(); });

```

* **Внешний вид интерфейса:** [Посмотреть скриншот](https://ibb.co/LdYKSmKr)

### Прототип вызова из Pawn

```pawn
// Параметры: nickname, id, name, surname, patronymic, level, cash, bank, frac, rang, food, drink, address, warns, blockVU, mute, jail
showAdminpanel('Nikolai_Zenetti', 15, 'Николай', 'Зенетти', 'Александрович', 17, 1500000, 4500000, 'Правительство', 'Губернатор', 85, 90, 'Арзамас, ул. Ленина, д. 1', 1, 0, 600, 1800);

```

---

## 5. Государственные Системы

### 🚔 Цифровой Алкотестер

Синхронизирует данные прибора как для инспектора ДПС, так и для проверяемого водителя.

```javascript
uCef.addEvent('interface:set:alcotesterResponse', (isInspector, type) => { alcotesterSetResponse(isInspector, type); }); // Ответ состояния
uCef.addEvent('interface:set:showAlcotester', (isInspector) => { showAlcotester(isInspector); });                    // Инициализация
uCef.addEvent('interface:set:hideAlcotester', () => { hideAlcotester(); });                                          // Принудительное закрытие
uCef.addEvent('interface:set:alcotesterShowCheck', (type, img, dataCheck) => { alcotesterShowCheck(type, img, dataCheck); }); // Вызов чека

```

* **Внешний вид интерфейса:** [Посмотреть скриншот](https://ibb.co/RRN2ZZm)

#### 🔹 JS Сценарий работы и эмуляции:

```javascript
// Порядок тестирования со стороны Инспектора (ДПС)
showAlcotester(true);   // Экран прибора без кнопки "ДУТЬ"
alcotesterSwitch();     // Запуск питания и загрузка системы устройства

// Симуляция успешного выдоха гражданина (зажатие ALT):
alcotesterSetResponse(true, 'setDrunk;-10000'); // Экспресс-скрининг (Чистый результат)
alcotesterSetResponse(true, 'setDrunk;45000');  // Полный тест (Генерация промилле на основе переданного числа)

alcotesterSetPage('fulldata'); // Переключение на форму ввода данных протокола (ФИО, Гос. Номер)
alcotesterPechat();            // Печать чека и инициализация поля Canvas для росписи

// Порядок тестирования со стороны Гражданского
showAlcotester(false);         // Экран прибора с активной кнопкой взаимодействия
alcotesterSetPage('scrining');  // Перевод на интерактивный экран продувки

```

### 📸 Измерительный Комплекс Радар (КРИС-П / КОРДОН)

```javascript
showRadar(0); // Включение обычного радара (патрульный тип, цикличный замер скорости, аудиозапуск)
showRadar(1); // Включение стационарного радара/комплекса КРИС-П (измененная анимация и вид)

// Функция обновления игровых данных: setRadar(model, plate, speed, isFlash)
setRadar("Vaz 2114", "А777АА 77", 74, 0);          // Обычный замер в рамках ПДД (чистый звук)
setRadar("Mercedes-Benz E63", "Е666КХ 97", 142, 1); // Нарушение установленного лимита! Срабатывание вспышки + звук radar2

```

### 🏁 Измеритель светопропускаемости стекол (Тоник)

```javascript
showTonik(true, 180);
tonikPower(); // Активация питания устройства

// Вывод итоговых значений на экран прибора: setTableNum(mode, formattedString)
setTableNum('all', '0;7;5.;3'); // Замер прошел успешно: 75.3% (ГОСТ соблюден)
setTableNum('all', '0;0;5.;0'); // Нарушение: 05.0% (Пленка не проходит по ГОСТу)

```

---

## 6. Бизнес-Интерфейсы

### 🚗 Интерфейс Автосалона

```javascript
uCef.addEvent('interface:set:showAvtosalon', (isLeader) => { showAvtosalon(isLeader); }); // Открытие (isLeader: 1 - показать кнопку закупки для орг, 0 - скрыть)
uCef.addEvent('interface:set:hideAvtosalon', () => { hideAvtosalon(); });                 // Закрытие автосалона
uCef.addEvent('interface:set:ASsetCar', (...args) => { ASsetCar(...args); });             // Обновление ТТХ машины на экране
uCef.addEvent('interface:set:showAvtosalonButton', (text) => { showAvtosalonButton(text); }); // Показ плашки-подсказки взаимодействия
uCef.addEvent('interface:set:hideAvtosalonButton', () => { hideAvtosalonButton(); });
uCef.addEvent('interface:set:showAvtosalonNotice', (text, color) => { showAvtosalonNotice(text, color); }); // Вывод статус-уведомлений

```

* **Внешний вид интерфейса:** [Посмотреть скриншот](https://ibb.co/vxh0Pxhz)

#### 🔹 Примеры инициализации карточек авто:

```javascript
showAvtosalon(1);

// Параметры карточки: namecar, year, speed, fuel, engine, power, typefuel, rashod, rear, kpp, kuzov, price
ASsetCar('ВАЗ 2107', 2003, 125, 60, 1529, '92/84', 'АИ-92', 8.5, 'Задний', 'МКПП', 'Седан', 125000);
ASsetCar('Tesla Model S', 2022, 250, 100, 0, '500/670', 'Электро', 15.2, 'Полный', 'АКПП', 'Лифтбек', 8500000);

// Вызов внутренней ошибки интерфейса
showAvtosalonNotice('Недостаточно средств на балансе!', 'red');

```

### ⛽ Заправочный Комплекс (АЗС)

* **Макеты интерфейсов:** [АЗС Топливо](https://ibb.co/0pQ49r6w) | [Электрозаправка](https://ibb.co/j9NNNy1c)

```javascript
// --- ТЕСТ СТАНДАРТНОЙ ТОПЛИВНОЙ АЗС ---
showAzsmenu();
setAzsmenuPage('oplata', 25.0, 60, 55); // Параметры: страница, цена_за_литр, литраж, текущий бак

// --- ТЕСТ СТАНЦИИ ЭЛЕКТРОЗАПРАВКИ ---
hideAzsmenu(); // Скрываем старое меню
showAzsmenuCharge(40.0, 15); // Параметры: цена за кВт, текущая емкость аккумулятора (%)

```

### 🏦 Меню Банкомата

```javascript
// Инициализация базового состояния банкомата
showBankomat(0, 'Nikolai_Zenetti', 5555, '4276 5500 1234 5678', 250);

// Вкладка налоговых обязательств
bankomatResponse('nalogi', 'houses', 'Дом №412', 'Налог: 1.500 ₽', 412);
bankomatResponse('nalogi', 'cars', 'Mercedes-Benz E63', 'Налог: 850 ₽', 77);

// Выбор конкретного объекта для оплаты
bankomatLastNalog = ['house', 412];
bankomatResponse('nalogiChoose', '3 дн.', 1500);

// Вкладка штрафов ГИБДД/ЦОРДД (Формат строки: ID/Причина/Дата/Статья/Сумма)
bankomatResponse('ticket', '1054/Превышение скорости/15.05.2026/КоАП 12.9/5000');
bankomatResponse('ticket', '1055/Встречная полоса/16.05.2026/КоАП 12.15/15000');

// Вкладка ипотечного кредитования
bankomatLastIpoteka = 12;
bankomatResponse('ipoteka', 'Квартира/12/5000000', '450000/5400000', '800000/4800000', '1500000/4500000');

```

### 💰 Интерфейс Автовыкупа / Свалки ТС

```javascript
// Параметры: имя_нпс, ид_транспорта, тип_объекта, цена_база, износ_двиг, износ_кузов, налог1, налог2, штрафы, скидка, итог_гос, итог_выкуп, дней_в_собств, гос_стоимость, страховка, коэф
showVikup('Никита', 962, 'Машина', 1500000, 1000, 5000, 150000, 200000, 20000, 10000, 25000, 670000, 5, 500000, 0, 3);

```

### 🎟️ Дополнительные уникальные вызовы

* **Донат-интерфейс заказа кастомных номеров:** `showDonateNomer();`

```

```
