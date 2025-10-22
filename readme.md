
# Тема: «Мини-маркетплейс товаров»

## Цель

Смоделировать простейший маркетплейс: товары, корзина, скидки, хранилище товаров. Потренироваться в проектировании иерархий, интерфейсов и коллекций.

---

## Ограничения

* Можно и нужно использовать `List`, `Set`, `Map`, циклы `for/while`, `if/else`, `switch`.

---

## 1) Интерфейсы

1. `Identifiable`

```java
import java.util.UUID;

public interface Identifiable {
    UUID getId(); // уникальный ID
}
```

2. `Priced`

```java
public interface Priced {
    int getPriceInCents(); // цена в копейках/центах (целое число)
}
```

3. `Discountable`

```java
public interface Discountable {
    /**
     * Применить скидку и вернуть новую цену в копейках.
     * discountPercent ∈ [0;100]
     */
    int applyDiscountPercent(int discountPercent);
}
```

---

## 2) Абстрактный класс товара

`AbstractProduct` — базовый класс для всех товаров.

* Реализует `Identifiable`, `Priced`, `Discountable`.
* Поля (protected/private + геттеры):

    * `String id`
    * `String name`
    * `int basePriceInCents`
* Конструктор с проверками (некорректные данные — `IllegalArgumentException`).
* Абстрактный метод:

  ```java
  public abstract String getCategory(); // например, "food", "electronics"
  ```
* `equals`/`hashCode` — по `id`.
* `toString()` — краткая информация о товаре.

---

## 3) Конкретные товары

Сделайте **минимум два** класса, наследующих `AbstractProduct`, например:

1. `FoodProduct`

* Доп. поле: `int shelfLifeDays` (срок годности в днях, >0).
* Категория: `"food"`.

2. `ElectronicProduct`

* Доп. поля: `int warrantyMonths` (гарантия в мес., ≥0).
* Категория: `"electronics"`.

Оба класса валидируют входящие параметры конструктора.

---

## 4) Корзина

Класс `Cart` — управляет позициями.

Вложенный класс `CartLine` (или отдельный публичный класс), поля:

* `AbstractProduct product`
* `int quantity` (≥1)

`Cart` хранит позиции в `List<CartLine>`.

Методы `Cart`:

```java
public void addProduct(AbstractProduct product, int quantity);
// Если товар уже есть — увеличить количество.

public boolean removeProductById(String productId);
// Удалить позицию по id товара. Вернуть true, если удалили.

public int getTotalInCents();
// Общая стоимость без скидок.

public int getTotalWithPercentDiscountInCents(int discountPercent);
// Применить одинаковую процентную скидку ко всем позициям (через Discountable).

public int getUniqueProductsCount();
// Кол-во уникальных товаров в корзине.

public Set<String> getCategories();
// Множество категорий товаров в корзине.

public void clear();
// Очистить корзину.
```

Валидации: `product != null`, `quantity >= 1`, `discountPercent` в [0;100]. Неверные данные — `IllegalArgumentException`.

---

## 5) Хранилище товаров (репозиторий)

`ProductRepository`

* Внутри `Map<String, AbstractProduct> byId = new HashMap<>();`
* Методы:

```java
public void save(AbstractProduct product);
// добавляет/перезаписывает по id

public AbstractProduct findById(String id); // вернуть товар или null

public boolean exists(String id);

public boolean delete(String id);

public List<AbstractProduct> findAll();
// вернуть копию списка

public List<AbstractProduct> findByCategory(String category);
// без стримов: обычный цикл, собираем подходящие в List
```

---

## 6) Сервис маркетплейса

`MarketplaceService`

* Поля: `ProductRepository repo`, `Cart cart`.
* Методы:

```java
public void addNewProduct(AbstractProduct product);
public AbstractProduct getProduct(String id);
public List<AbstractProduct> listAll();
public List<AbstractProduct> listByCategory(String category);

public void addToCart(String productId, int quantity);
// найти товар в repo и добавить в cart

public int cartTotal();
public int cartTotalWithDiscount(int discountPercent);
public void cartClear();
```

Ошибки (например, товар не найден) — `IllegalStateException` или `IllegalArgumentException` с понятным сообщением.

---

## 7) Консольный `Main`

* Создать `ProductRepository`, `Cart`, `MarketplaceService`.
* Добавить 3–5 товаров разных типов.
* Показать:

    * список всех товаров,
    * добавление в корзину (2–3 позиции),
    * вывод суммы без скидки и со скидкой (например, 10%),
    * вывод категорий в корзине,
    * удаление позиции и повторный пересчет,
    * очистку корзины.

Вывод должен быть понятным (несколько `System.out.println()` с подписями).

---

## 8) Пример сценария (как должно работать)

```
Добавлены товары: p1, p2, p3
Все товары (3): [FoodProduct{id=p1,...}, ElectronicProduct{id=p2,...}, FoodProduct{id=p3,...}]

Добавляю в корзину: p1 x2, p2 x1
Итого без скидки: 125000 (в копейках)
Итого со скидкой 10%: 112500
Категории в корзине: [food, electronics]
Уникальных товаров: 2

Удаляю p2 -> true
Итого без скидки: 100000
Очищаю корзину...
Корзина пуста
```

(Цифры произвольны; важна логика.)