from typing import Tuple

# Základní třída pro všechny objekty v hře
class Entity:
    def __init__(self, name: str, coords: Tuple[int, int], hit_points: int) -> None:
        # Inicializace základních atributů entity
        self.name = name  # Název entity
        self.coords = coords  # Souřadnice entity jako (x, y)
        self.hit_points = hit_points  # Počet životů entity

    def take_damage(self, damage_amount: int) -> None:
        # Tato metoda je abstraktní a musí být implementována v podtřídách
        raise NotImplementedError("This method is abstract. Please implement it")

    def is_alive(self) -> bool:
        # Zkontroluje, jestli má entita ještě životy
        return self.hit_points > 0

    def get_distance(self, other_coords: Tuple[int, int]) -> int:
        # Vypočítá vzdálenost od jiné entity pomocí Manhattan metriky (taxicab distance)
        return abs(self.coords[0] - other_coords[0]) + abs(self.coords[1] - other_coords[1])

# Třída pro kameny, které jsou odolné vůči slabým útokům
class Rock(Entity):
    def take_damage(self, damage_amount: int) -> None:
        # Kámen přijímá poškození pouze, pokud je poškození >= 10
        # Pokud ano, ztratí 1 hitpoint
        if damage_amount >= 10:
            self.hit_points -= 1

# Třída pro nábytek, který přijímá poškození plně
class Furniture(Entity):
    def take_damage(self, damage_amount: int) -> None:
        # Nábytek ztrácí tolik hitpointů, kolik je poškození
        self.hit_points -= damage_amount

# Třída pro živé bytosti (postavy, které mají level, útok, pohyb, atd.)
class LivingEntity(Entity):
    def __init__(self, name: str, coords: Tuple[int, int], hit_points: int, level: int, damage: int, attack_range: int) -> None:
        # Inicializace pomocí základní třídy Entity a přidání dalších atributů specifických pro živé entity
        super().__init__(name, coords, hit_points)
        self.level = level  # Úroveň entity
        self.damage = damage  # Poškození, které entity způsobí
        self.attack_range = attack_range  # Dostřel útoku

    def level_up(self) -> None:
        # Zvýšení úrovně entity o 1
        self.level += 1

    def take_damage(self, damage_amount: int) -> None:
        # Živá bytost ztrácí tolik hitpointů, kolik je poškození
        self.hit_points -= damage_amount

    def hit(self, other_entity: Entity) -> None:
        # Tato metoda je abstraktní a musí být implementována v podtřídách
        raise NotImplementedError("This method is abstract. Please implement it")

    def move(self, vector: Tuple[int, int]) -> None:
        # Pohybuje entitou podle vektoru (x, y)
        self.coords = (self.coords[0] + vector[0], self.coords[1] + vector[1])

    def in_range(self, other_entity: Entity) -> bool:
        # Zkontroluje, jestli je jiná entita v dostřelu pro útok
        return self.get_distance(other_entity.coords) <= self.attack_range

# Třída pro válečníky, kteří mají vyšší poškození a zvláštní mechaniku pro přijímání poškození
class Warrior(LivingEntity):
    def hit(self, other_entity: Entity) -> None:
        # Pokud je entita v dostřelu, způsobí poškození
        if self.in_range(other_entity):
            other_entity.take_damage(self.damage)

    def take_damage(self, damage_amount: int) -> None:
        # Válečník přijímá poškození pouze tehdy, pokud je větší než jeho úroveň
        if damage_amount > self.level:
            self.hit_points -= damage_amount - self.level

    def level_up(self) -> None:
        # Zvýšení úrovně válečníka, zlepšení poškození a hitpointů
        super().level_up()  # Volání metody level_up z nadtřídy
        self.damage += 1  # Zvětšení poškození o 1
        self.hit_points += 2  # Zvětšení hitpointů o 2

# Třída pro lukostřelce, kteří mají velký dostřel a zvyšují poškození o více než válečníci
class Archer(LivingEntity):
    def hit(self, other_entity: Entity) -> None:
        # Pokud je entita v dostřelu, způsobí poškození
        if self.in_range(other_entity):
            other_entity.take_damage(self.damage)

    def level_up(self) -> None:
        # Zvýšení úrovně lukostřelce, zlepšení poškození a hitpointů
        super().level_up()  # Volání metody level_up z nadtřídy
        self.damage += 2  # Zvětšení poškození o 2
        self.hit_points += 1  # Zvětšení hitpointů o 1

# Testy pro ověření správnosti implementace

# Test pro Entity
entity = Entity("E", (0, 0), 10)
assert entity.name == "E"
assert entity.coords == (0, 0)
assert entity.hit_points == 10
assert entity.is_alive()  # Entita je naživu
assert entity.get_distance((0, 0)) == 0  # Vzdálenost k sobě je 0

# Test pro Rock (kámen)
rocky = Rock("Rocky", (0, 1), 5)
assert rocky.name == "Rocky"
assert rocky.coords == (0, 1)
assert rocky.hit_points == 5
assert rocky.is_alive()  # Kámen je naživu
assert rocky.get_distance((0, 0)) == 1  # Vzdálenost k (0, 0) je 1
rocky.take_damage(7)  # Poškození menší než 10
assert rocky.hit_points == 5  # Žádné poškození
rocky.take_damage(100)  # Poškození větší než 10
assert rocky.hit_points == 4  # Poškození 1 hitpoint

# Test pro Furniture (nábytek)
table = Furniture("Table", (1, 0), 3)
assert table.name == "Table"
assert table.coords == (1, 0)
assert table.hit_points == 3
assert table.is_alive()  # Stůl je naživu
assert table.get_distance((0, 0)) == 1  # Vzdálenost k (0, 0) je 1
table.take_damage(2)
assert table.hit_points == 1  # Poškození 2
table.take_damage(2)
assert table.hit_points == -1  # Poškození 2
assert not table.is_alive()  # Stůl je mrtvý

# Test pro LivingEntity (živou bytost)
tim = LivingEntity("Tim", (-1, 0), 3, 1, 1, 1)
assert tim.name == "Tim"
assert tim.coords == (-1, 0)
assert tim.hit_points == 3
assert tim.level == 1
assert tim.damage == 1
assert tim.attack_range == 1
tim.level_up()  # Zlepšení úrovně
assert tim.level == 2
tim.take_damage(1)
assert tim.hit_points == 2  # Ztráta 1 hitpointu
tim.move((0, -1))  # Pohyb
assert not tim.in_range(entity)  # Není v dosahu

# Test pro Warrior (válečník)
willy = Warrior("William Wallace", (1, 1), 3, 5, 8, 2)
assert willy.name == "William Wallace"
assert willy.coords == (1, 1)
assert willy.hit_points == 3
assert willy.level == 5
assert willy.damage == 8
assert willy.attack_range == 2
willy.level_up()  # Zlepšení úrovně válečníka
assert willy.hit_points == 5
assert willy.level == 6
assert willy.damage == 9
rocky = Rock("Rocky", (0, 1), 5)
willy.hit(rocky)  # Válečník útočí na kámen (nepřijme poškození)
assert rocky.hit_points == 5

# Test pro Archer (lukostřelec)
rob = Archer("Robin Hood", (5, 8), 3, 1, 7, 10)
assert rob.name == "Robin Hood"
assert rob.coords == (5, 8)
assert rob.hit_points == 3
assert rob.level == 1
assert rob.damage == 7
assert rob.attack_range == 10
rob.level_up()  # Zlepšení úrovně lukostřelce
assert rob.hit_points == 4
assert rob.level == 2
assert rob.damage == 9
willy = Warrior("William Wallace", (1, 1), 3, 5, 8, 2)
rob.hit(willy)  # Lukostřelec útočí na válečníka (přijme poškození)
assert willy.hit_points == 3
rob.move((0, -1))  # Pohyb
rob.hit(willy)  # Lukostřelec znovu útočí na válečníka
assert willy.hit_points == -1  # Válečník je mrtvý
