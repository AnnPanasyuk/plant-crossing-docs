classDiagram
class User {
+int id
+string username
+string wallet_address
+int level
+float balance
+last_login_at
}

    class Plant {
        +int id
        +string name
        +string rarity
        +int power
        +int speed
        +int stamina
        +string status
    }

    class UserPlant {
        +int id
        +int user_id
        +int plant_id
        +datetime acquired_at
        +int current_level
        +int experience
    }

    class ActionLog {
        +int id
        +int user_id
        +string action_type
        +datetime timestamp
    }

    class Marketplace {
        +int id
        +int plant_id
        +int seller_id
        +float price
        +string status
    }

    User "1" -- "*" UserPlant : owns
    Plant "1" -- "*" UserPlant : template_for
    User "1" -- "*" ActionLog : performs
    User "1" -- "*" Marketplace : lists
    UserPlant "1" -- "0..1" Marketplace : listed_in

