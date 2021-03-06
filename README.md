# Questions:

**How many users are there?**

50
 ```sql
 select count(*) from users;
 ```

**What are the 5 most expensive items?**

* Small Cotton Gloves  9984
* Small Wooden Comput  9859
* Awesome Granite Pan  9790
* Sleek Wooden Hat     9390
* Ergonomic Steel Car  9341

```sql
select title, price from items order by price desc limit 5;
```

**What's the cheapest book? (Does that change for "category is exactly 'book'" versus "category contains 'book'"?)**

Ergonomic Granite Chair  1496 and no it doesn't change.

```sql
select category, title, min(price) from items where category like '%Books%';

select category, title, min(price) from items where category = '%Books%';
```

**Who lives at "6439 Zetta Hills, Willmouth, WY"? Do they have another address?**

Corrine     Little      6439 Zetta Hills  Willmouth   WY

Corrine     Little      54369 Wolff Forg  Lake Bryon  CA
```sql
select first_name, last_name, street, city, state from users u left join addresses a on u.id = a.user_id where u.id = (select user_id from addresses a where a.street = '6439 Zetta Hills' and a.city = 'Willmouth' and a.state = 'WY');
```

**Correct Virginie Mitchell's address to "New York, NY, 10108".**


```sql
update addresses set city = 'New York', state = 'NY', zip = '10108' where id = (select id from users where first_name = 'Virginie' and last_name =  'Mitchell');
```

**How much would it cost to buy one of each tool?**

7383.0
```sql
select total(price) from items where category = 'Tools';
```

**How many total items did we sell?**

2125
```sql
select sum(quantity) from orders;
```

**How much was spent on books?**

420566.0
```sql
select total(i.price * o.quantity) from orders o inner join items i on o.item_id = i.id where i.category = 'Books';
```


**Simulate buying an item by inserting a User for yourself and an Order for that User.**


```sql
insert into users (first_name, last_name, email) values ('Tom', 'McClelland', 'tom@mcclelland.me');

select id from users where email = 'tom@mcclelland.me';

select id from items where title = 'Gorgeous Cotton Pants';

insert into orders (user_id, item_id, quantity, created_at) values (51, 42, 365, datetime());
```


# Adventure Mode

**What item was ordered most often? Grossed the most money?**

Ordered most: Ergonomic Concrete Gloves  9

Grossed most: Gorgeous Cotton Pants  1028935

```sql
select orders.item_id, items.title, max(itemCount) from orders
inner join
    (select item_id, count(item_id) as itemCount from orders
    group by item_id
    order by count(item_id) desc) groupedOrders    
on orders.item_id = groupedOrders.item_id
inner join items on orders.item_id = items.id;

select orders.item_id, items.title, max(total) from orders
inner join
    (select o.item_id, i.price * o.quantity as total from orders o inner join items i on o.item_id = i.id order by total desc)
     groupedOrders    
on orders.item_id = groupedOrders.item_id
inner join items on orders.item_id = items.id;
```

**What user spent the most?**

Tom         McClelland  1028935

```sql
select u.id, u.first_name, u.last_name, max(total) from users u
inner join
    (select u.id, u.first_name, u.last_name, i.price * o.quantity as total
    from orders o
    inner join items i on o.item_id = i.id
    inner join users u on o.user_id = u.id
    order by total desc) groupedUsers    
on u.id = groupedUsers.id;
```

**What were the top 3 highest grossing categories?**

Computers	1028935

Toys & Books	97900

Health & Grocery	91290

```sql
select i.id, category, i.price * o.quantity as total
    from orders o
    inner join items i on o.item_id = i.id
    order by total desc limit 3;
```
