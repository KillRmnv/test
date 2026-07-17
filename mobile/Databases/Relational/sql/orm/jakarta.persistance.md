
Ниже — основные аннотации Jakarta Persistence (бывший javax.persistence), применяемые для ORM в Java.

---

# 📌 Основные аннотации JPA (Jakarta Persistence)

## **1. Аннотации для сущностей**

|Аннотация|Назначение|
|---|---|
|`@Entity`|Помечает класс как сущность ORM.|
|`@Table(name = "...")`|Определяет таблицу в БД, к которой привязана сущность.|
|`@MappedSuperclass`|Базовый класс, чьи поля наследуются сущностями.|
|`@Embeddable`|Класс-встраиваемый тип.|
|`@Embedded`|Использовать встраиваемый объект внутри сущности.|

---

## **2. Поля и колонки**

|Аннотация|Назначение|
|---|---|
|`@Id`|Первичный ключ.|
|`@GeneratedValue(strategy = ...)`|Генерация PK (IDENTITY, SEQUENCE, AUTO, TABLE).|
|`@Column(name = "...")`|Настройка колонки.|
|`@Transient`|Поле не сохраняется в БД.|
|`@Version`|Версионирование для Optimistic Locking.|

---

## **3. Отношения между сущностями**

|Аннотация|Назначение|
|---|---|
|`@OneToOne`|Связь один-к-одному.|
|`@OneToMany`|Связь один-ко-многим.|
|`@ManyToOne`|Связь много-к-одному.|
|`@ManyToMany`|Связь многие-ко-многим.|
|`@JoinColumn(name = "...")`|Настройка внешнего ключа.|
|`@JoinTable(...)`|Промежуточная таблица (для ManyToMany).|

---

## **4. Аннотации для JPQL и SQL**

|Аннотация|Назначение|
|---|---|
|`@NamedQuery`|Статический JPQL-запрос.|
|`@NamedQueries`|Группа запросов.|
|`@NamedNativeQuery`|Статический SQL-запрос.|
|`@SqlResultSetMapping`|Маппинг результата SQL.|

---

## **5. Дополнительные**

|Аннотация|Назначение|
|---|---|
|`@Enumerated(EnumType.STRING/ORDINAL)`|Хранение Enum.|
|`@Lob`|Большие объекты: TEXT/CLOB/BLOB.|
|`@Convert(converter = ...)`|Подключение собственного конвертера типа.|
|`@CreationTimestamp` / `@UpdateTimestamp`|Для автоматического времени создания/обновления (Hibernate).|

---

# 📦 Пример минимальной сущности JPA

```java
import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @ManyToOne
    @JoinColumn(name = "role_id")
    private Role role;
}
```
