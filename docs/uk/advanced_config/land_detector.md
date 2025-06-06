# Конфігурація детектора посадки

Детектор посадки - динамічна модель транспортного засобу, що представляє ключові стани транспортного засобу від контакту з землею до посадки.
Ця тема пояснює основні параметри, які ви, можливо, захочете налаштувати для покращення поведінки при посадці.

## Автоматичне вимкнення

Детектор посадки автоматично вимикає управління дроном при посадці.

Ви можете встановити [COM_DISARM_LAND](../advanced_config/parameter_reference.md#COM_DISARM_LAND) , щоб вказати кількість секунд після посадки, протягом яких система повинна вимкнутися (або вимкнути автоматичне вимикання, встановивши параметр на -1).

## Конфігурація мультикоптера

Повний набір відповідних параметрів детектора приземлення перераховано в посиланні на параметри з префіксом [LNDMC](../advanced_config/parameter_reference.md#land-detector) (їх можна редагувати в QGroundControl через [редактор параметрів](../advanced_config/parameters.md)).

:::tip
Інформація про те, як параметри впливають на посадку, може бути знайдена нижче в [Стани виявлення посадки](#mc-land-detector-states).
:::

Інші ключові параметри, які вам може знадобитися налаштувати для покращення поведінки приземлення на конкретних повітряних суднах:

- [MPC_THR_HOVER](../advanced_config/parameter_reference.md#MPC_THR_HOVER) - дросельна заслінка системи (за замовчуванням 50%).
  Важливо правильно встановити це значення, оскільки це робить управління висотою більш точним та гарантує правильне виявлення посадки.
  Гоночний або великий квадрокоптер для зйомки без встановленого вантажу може потребувати набагато нижчого налаштування (наприклад, 35%).

  Неправильне встановлення параметра `MPC_THR_HOVER` може призвести до виявлення контакту з землею або, можливо, приземлення, коли транспортний засіб все ще знаходиться в повітрі (зокрема, під час спуску в режимі [Позиції](../flight_modes_mc/position.md) або [Висоти](../flight_modes_mc/altitude.md)).
  Це призводить до "тремтіння" транспортного засобу (зниження обертів двигуна, а потім негайне збільшення їх).

:::

- [MPC_THR_MIN](../advanced_config/parameter_reference.md#MPC_THR_MIN) - загальний мінімум газу системи.
  Це повинно бути встановлено для забезпечення контрольного спуску.

- [MPC_LAND_CRWL](../advanced_config/parameter_reference.md#MPC_LAND_CRWL) - вертикальна швидкість, яка застосовується в останній стадії автономного приземлення, якщо система має датчик відстані, і він присутній і працює. Повинен бути встановлений більшим, ніж LNDMC_Z_VEL_MAX.

### MC Мультикоптер детектор землі

Для виявлення посадки багтороте повітряне судно спочатку повинно пройти три різні стани, кожен з яких містить умови попередніх станів плюс більш жорсткі обмеження.
Якщо умова не може бути досягнута через відсутність сенсорів, то умова, за замовчуванням, є істинною.
Наприклад, у режимі [Acro](../flight_modes_mc/acro.md), коли жоден сенсор, крім гіроскопа, не активний, виявлення базується лише на виводі потужності і часі.

Для переходу до наступного стану кожна умова повинна бути виконана протягом однієї третини загального налаштованого часу спрацювання детектора посадки [LNDMC_TRIG_TIME](../advanced_config/parameter_reference.md#LNDMC_TRIG_TIME).
Якщо транспортний засіб обладнаний датчиком відстані, але відстань до землі в даний момент не вимірюється (зазвичай через те, що вона занадто велика), час спрацювання збільшується втричі.

Якщо одна умова не виконується, виявник посадки негайно виходить з поточного стану.

#### Контакт із землею

Якщо одна умова не виконується, виявник посадки негайно виходить з поточного стану.

- відсутність вертикального руху ([LNDMC_Z_VEL_MAX](../advanced_config/parameter_reference.md#LNDMC_Z_VEL_MAX))
- немає горизонтального руху ([LNDMC_XY_VEL_MAX](../advanced_config/parameter_reference.md#LNDMC_XY_VEL_MAX))
- нижча тяга, ніж [MPC_THR_MIN](../advanced_config/parameter_reference.md#MPC_THR_MIN) + (дросель при зависанні - [MPC_THR_MIN](../advanced_config/parameter_reference.md#MPC_THR_MIN)) \* (0,3, якщо оцінка тяги зависання недоступна, тоді 0,6),
- додаткова перевірка, чи зараз транспортний засіб перебуває в режимі польоту з контрольованою висотою: транспортний засіб має мати намір знизитися (задане значення вертикальної швидкості вище LNDMC_Z_VEL_MAX).
- додаткова перевірка для автомобілів з датчиком відстані: поточна відстань до землі менше 1 м.

Умови для цього стану:

#### Може бути приземлена

Якщо одна умова не виконується, виявник посадки негайно виходить з поточного стану.

- всі умови стану [контакту з землею](#ground-contact) виконані
- не обертається ([LNDMC_ROT_MAX](../advanced_config/parameter_reference.md#LNDMC_ROT_MAX))
- має низький тяговий потік `MPC_THR_MIN + (MPC_THR_HOVER - MPC_THR_MIN) * 0.1`
- вільного падіння не виявлено

Умови для цього стану:

Якщо транспортний засіб перебуває у керуванні положенням або швидкістю, і, можливо, виявлено приземлення,
контролер положення встановить вектор тяги на нуль.

#### Посадка

Якщо одна умова не виконується, виявник посадки негайно виходить з поточного стану.

- усі умови стану [maybe landed](#maybe-landed) вірні

## Конфігурація фіксованого крила

Параметри налаштування для визначення посадки фіксованим крилом:

- [LNDFW_AIRSPD_MAX](../advanced_config/parameter_reference.md#LNDFW_AIRSPD_MAX) - Максимальна швидкість повітря, яка дозволяє системі все ще вважатися приземленою.
  Це повинен бути компроміс між точністю вимірювання швидкості повітря та швидкістю спрацювання.
  Кращі датчики швидкості повітря дозволяють встановлювати менші значення цього параметра.
- [LNDFW_VEL_XY_MAX ](../advanced_config/parameter_reference.md#LNDFW_VEL_XY_MAX) - Максимальна горизонтальна швидкість, при якій система все ще вважається приземленою.
- [LNDFW_VEL_Z_MAX](../advanced_config/parameter_reference.md#LNDFW_VEL_XY_MAX) - Максимальна вертикальна швидкість, при якій система все ще вважається приземленою.
- [LNDFW_XYACC_MAX](../advanced_config/parameter_reference.md#LNDFW_XYACC_MAX) - Максимальне горизонтальне прискорення, при якому система все ще вважається приземленою.
- [LNDFW_TRIG_TIME](../advanced_config/parameter_reference.md#LNDFW_TRIG_TIME) - Час спрацювання, протягом якого вищезазначені умови повинні бути виконані для оголошення посадки.

:::info
Коли виявлення запуску FW увімкнено ([FW_LAUN_DETCN_ON](../advanced_config/parameter_reference.md#FW_LAUN_DETCN_ON)), транспортний засіб залишиться в стані "приземлено" до виявлення зльоту (що базується виключно на прискоренні, а не на швидкості).
:::

## VTOL детектор землі

Детектор землі VTOL 1:1 такий самий, як детектор землі MC, якщо система знаходиться в режимі висіння. У режимі FW виявлення землі вимкнено.
