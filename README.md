#How many users are there?

[2] pry(main)> User.count
   (0.3ms)  SELECT COUNT( * ) FROM "users"
=> 51

My DB from yesterday has my orders for me in it still.

#What are the 5 most expensive items?

[14] pry(main)> Item.order(price: :desc).select("id, title, price").limit(5)
  Item Load (0.4ms)  SELECT  id, title, price FROM "items"  ORDER BY "items"."price" DESC LIMIT 5
=> [#<Item:0x007f96459a4f88 id: 25, title: "Small Cotton Gloves", price: 9984>,
 #<Item:0x007f96459a4cb8 id: 83, title: "Small Wooden Computer", price: 9859>,
 #<Item:0x007f96459a49c0 id: 100, title: "Awesome Granite Pants", price: 9790>,
 #<Item:0x007f96459a4858 id: 40, title: "Sleek Wooden Hat", price: 9390>,
 #<Item:0x007f964599ffb0 id: 60, title: "Ergonomic Steel Car", price: 9341>]

#What's the cheapest book?

[16] pry(main)> Item.order(price: :asc).where(category: "Books").select("id, title, price").limit(1)
  Item Load (0.4ms)  SELECT  id, title, price FROM "items" WHERE "items"."category" = ?  ORDER BY "items"."price" ASC LIMIT 1  [["category", "Books"]]
=> [#<Item:0x007f964584b8a8 id: 76, title: "Ergonomic Granite Chair", price: 1496>]

#Who lives at "6439 Zetta Hills, Willmouth, WY"? Do they have another address?

[67] pry(main)> User.find(Address.where(street: "6439 Zetta Hills").first.user_id)
  Address Load (0.2ms)  SELECT  "addresses".* FROM "addresses" WHERE "addresses"."street" = ?  ORDER BY "addresses"."id" ASC LIMIT 1  [["street", "6439 Zetta Hills"]]
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT 1  [["id", 40]]
=> #<User:0x007f9644458c38 id: 40, first_name: "Corrine", last_name: "Little", email: "rubie_kovacek@grimes.net">



[66] pry(main)> Address.order("user_id").where( user_id: (Address.where(street: "6439 Zetta Hills").first.user_id))
  Address Load (0.2ms)  SELECT  "addresses".* FROM "addresses" WHERE "addresses"."street" = ?  ORDER BY "addresses"."id" ASC LIMIT 1  [["street", "6439 Zetta Hills"]]
  Address Load (0.3ms)  SELECT "addresses".* FROM "addresses" WHERE "addresses"."user_id" = ?  ORDER BY user_id  [["user_id", 40]]
=> [#<Address:0x007f9644530cc8 id: 43, user_id: 40, street: "6439 Zetta Hills", city: "Willmouth", state: "WY", zip: 15029>,
 #<Address:0x007f9644530b10 id: 44, user_id: 40, street: "54369 Wolff Forges", city: "Lake Bryon", state: "CA", zip: 31587>]

#Correct Virginie Mitchell's address to "New York, NY, 10108".

I am using the already updated database from previous work, so I will be moving her to Hawaii, she deserves a break.
However, it is a new summer home, but to allow me to use Update instead of Create, the 'agent' putting it in made a mistake
the first try.

[19] pry(main)> Address.create(user_id: user, street: "5162 Fancy Beach", city: "Crystal Shores", state: "FL", zip: 90210)
   (0.1ms)  begin transaction
  SQL (0.5ms)  INSERT INTO "addresses" ("user_id", "street", "city", "state", "zip") VALUES (?, ?, ?, ?, ?)  [["user_id", 39], ["street", "5162 Fancy Beach"], ["city", "Crystal Shores"], ["state", "FL"], ["zip", 90210]]
   (39.5ms)  commit transaction
=> #<Address:0x007f9642c069c8 id: 55, user_id: 39, street: "5162 Fancy Beach", city: "Crystal Shores", state: "FL", zip: 90210>

[27] pry(main)> address = Address.find_by(id: 55)
  Address Load (0.2ms)  SELECT  "addresses".* FROM "addresses" WHERE "addresses"."id" = ? LIMIT 1  [["id", 55]]
=> #<Address:0x007f9641b711a8 id: 55, user_id: 39, street: "5162 Fancy Beach", city: "Crystal Shores", state: "FL", zip: 90210>
[28] pry(main)> address.update(state: "HI", zip: 98682)
   (0.1ms)  begin transaction
  SQL (0.8ms)  UPDATE "addresses" SET "state" = ?, "zip" = ? WHERE "addresses"."id" = ?  [["state", "HI"], ["zip", 98682], ["id", 55]]
   (39.1ms)  commit transaction
=> true
[29] pry(main)> Address.where(id: 55)
  Address Load (0.2ms)  SELECT "addresses".* FROM "addresses" WHERE "addresses"."id" = ?  [["id", 55]]
=> [#<Address:0x007f9641b013f8 id: 55, user_id: 39, street: "5162 Fancy Beach", city: "Crystal Shores", state: "HI", zip: 98682>]
[30] pry(main)>



#How much would it cost to buy one of each tool?

[10] pry(main)> Item.where(category: "Tools").sum("price")
   (0.2ms)  SELECT SUM("items"."price") FROM "items" WHERE "items"."category" = ?  [["category", "Tools"]]
=> 7383

#How many total items did we sell?

[7] pry(main)> Order.sum("quantity")
   (0.6ms)  SELECT SUM("orders"."quantity") FROM "orders"
=> 2133

#How much was spent on books?

[1] pry(main)> book_ids = Item.where(category: "Books").ids
   (1.2ms)  SELECT "items"."id" FROM "items" WHERE "items"."category" = ?  [["category", "Books"]]
=> [4, 21, 76, 98]

[5] pry(main)> total = 0
=> 0
[6] pry(main)> book_ids.each do |x|
[6] pry(main)*   total += (Order.where(item_id: x).sum("quantity") * Item.where(id: x).first.price)
[6] pry(main)* end
   (0.3ms)  SELECT SUM("orders"."quantity") FROM "orders" WHERE "orders"."item_id" = ?  [["item_id", 4]]
  Item Load (0.2ms)  SELECT  "items".* FROM "items" WHERE "items"."id" = ?  ORDER BY "items"."id" ASC LIMIT 1  [["id", 4]]
   (0.2ms)  SELECT SUM("orders"."quantity") FROM "orders" WHERE "orders"."item_id" = ?  [["item_id", 21]]
  Item Load (0.2ms)  SELECT  "items".* FROM "items" WHERE "items"."id" = ?  ORDER BY "items"."id" ASC LIMIT 1  [["id", 21]]
   (0.2ms)  SELECT SUM("orders"."quantity") FROM "orders" WHERE "orders"."item_id" = ?  [["item_id", 76]]
  Item Load (0.2ms)  SELECT  "items".* FROM "items" WHERE "items"."id" = ?  ORDER BY "items"."id" ASC LIMIT 1  [["id", 76]]
   (0.2ms)  SELECT SUM("orders"."quantity") FROM "orders" WHERE "orders"."item_id" = ?  [["item_id", 98]]
  Item Load (0.2ms)  SELECT  "items".* FROM "items" WHERE "items"."id" = ?  ORDER BY "items"."id" ASC LIMIT 1  [["id", 98]]
=> [4, 21, 76, 98]
[7] pry(main)> total
=> 420566


#Simulate buying an item by inserting a User for yourself and an Order for that User.

[12] pry(main)> User.create(first_name: "Ruby", last_name: "Reicht", email: "noway@gotohell.com")
   (0.2ms)  begin transaction
  SQL (0.8ms)  INSERT INTO "users" ("first_name", "last_name", "email") VALUES (?, ?, ?)  [["first_name", "Ruby"], ["last_name", "Reicht"], ["email", "noway@gotohell.com"]]
   (0.7ms)  commit transaction
=> #<User:0x007f96444520e0 id: 52, first_name: "Ruby", last_name: "Reicht", email: "noway@gotohell.com">

[14] pry(main)> Order.create(user_id: 52, item_id: 57, quantity: 3)
   (0.1ms)  begin transaction
  SQL (0.5ms)  INSERT INTO "orders" ("user_id", "item_id", "quantity", "created_at") VALUES (?, ?, ?, ?)  [["user_id", 52], ["item_id", 57], ["quantity", 3], ["created_at", "2016-03-30 13:24:14.375028"]]
   (0.9ms)  commit transaction
=> #<Order:0x007f96419f7f20 id: 380, user_id: 52, item_id: 57, quantity: 3, created_at: Wed, 30 Mar 2016 13:24:14 UTC +00:00>
