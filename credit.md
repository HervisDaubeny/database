# Zápočtová práce na Databázové systémy - Sebastian Uhlík (uhlikse)

## Konceptuální modelování

### Popis
Můj informační systém budiž databáze pro webové stránky mojí táborové organizace. Systém slouží pro přihlašování dětí na táborové akce, pro evidenci plateb za akce a uložení přihlašovacích údajů uživatelů pro autentifikaci při přihlašování.

Obsahuje následující třídy:
```Parrent```, ```Child```, ```Event```, ```Payment```, ```User```, ```Admin```, ```Accountant```

Vazby v databázi jsou následující:
```
Parrent : Child = 1:N
Child : Event = M:N
Parrent : Payment = 1:N
Child : Payment = M:N
Event : Payment = M:N
```

### UML model:
![model](/UML.png)

### Relační model:
```Parrent(ParrentID,Name,Surname,Phone,Email,Address,Passwd)```  
```Child(ChildID,ParrentID,Name,Surname,Phone,Email,Address,Birth,Alergies)```  
$\qquad$```Child.ChildID``` $\subset$ ```Parrent.ParrentID```  
```Event(EventID,Type,Name,Begins,Ends,Capacity)```  
```Payment(PaymentID,ParrentID,Due,Date,Type)```  
$\qquad$```Payment.PaymentID``` $\subset$ ```Parrent.ParrentID```  
```User(UserID,Email,Passwd)```  
```Admin(AdminID,Email,Passwd)```  
```Accountant(AccountantID,Email,Passwd)```  
```GoesTo(ChildID,EventID)```  
$\qquad$```GoesTo.ChildID``` $\subset$ ```Child.ChildID```  
$\qquad$```GoesTo.EventID``` $\subset$ ```Event.EventID```  
```PaidForWhom(PaymentID,ChildID)```  
$\qquad$```PaidForWhom.PaymentID``` $\subset$ ```Payment.PaymentID```  
$\qquad$```PaidForWhom.ChildID``` $\subset$ ```Child.ChildID```  
```PaidForWhat(PaymentID,EventID)```  
$\qquad$```PaidForWhat.PaymentID``` $\subset$ ```Payment.PaymentID```  
$\qquad$```PaidForWhat.EventID``` $\subset$ ```Event.EventID```  

### Fyzický model:
```sql
CREATE TABLE Parrent (
ParrentID int IDENTITY(1,1) PRIMARY KEY,
Name VARCHAR(20) NOT NULL,
Surname VARCHAR(35) NOT NULL,
Phone int NOT NULL,
Email VARCHAR(50) NOT NULL,
Address VARCHAR(50) NOT NULL,
Passwd VARCHAR(20) NOT NULL
);
CREATE TABLE Child (
ChildID int IDENTITY(1,1) PRIMARY KEY,
ParrentID int FOREIGN KEY REFERENCES Parrent(ParrentID) ON DELETE CASCADE,
Name VARCHAR(20) NOT NULL,
Surname VARCHAR(25) NOT NULL,
Phone INT,
Email VARCHAR(50),
Address VARCHAR(50) NOT NULL,
Birth DATE NOT NULL,
Alergies VARCHAR(128)
);
CREATE TABLE Event (
EventID int IDENTITY(1,1) PRIMARY KEY,
Type VARCHAR(20) NOT NULL,
Name VARCHAR(50) NOT NULL,
Begins DATE NOT NULL,
Ends DATE NOT NULL,
Capacity int NOT NULL
);
CREATE TABLE Payment (
PaymentID int IDENTITY(1,1) PRIMARY KEY,
ParrentID int FOREIGN KEY REFERENCES Parrent(ParrentID) ON DELETE CASCADE,
Due int NOT NULL,
Date DATE NOT NULL,
Type VARCHAR(20) NOT NULL
); 
CREATE TABLE MyUser (
UserID int IDENTITY(1,1) PRIMARY KEY,
Email VARCHAR(50) NOT NULL,
Passwd VARCHAR(25) NOT NULL
);
CREATE TABLE Admin (
AdminID int IDENTITY(1,1) PRIMARY KEY,
Email VARCHAR(50) NOT NULL,
Passwd VARCHAR(25) NOT NULL
);
CREATE TABLE Accountant (
AccountatntID int IDENTITY(1,1) PRIMARY KEY,
Email VARCHAR(50) NOT NULL,
Passwd VARCHAR(25) NOT NULL
);
CREATE TABLE GoesTo (
ChildID int FOREIGN KEY REFERENCES Child(ChildID),
EventID int FOREIGN KEY REFERENCES Event(EventID) ON DELETE CASCADE
);
CREATE TABLE PaidForWhom (
PaymentID int FOREIGN KEY REFERENCES Payment(PaymentID),
ChildID int FOREIGN KEY REFERENCES Child(ChildID)
);
CREATE TABLE PaidForWhat (
PaymentID int FOREIGN KEY REFERENCES Payment(PaymentID) ON DELETE CASCADE,
EventID int FOREIGN KEY REFERENCES Event(EventID) ON DELETE CASCADE
);
```


## Dotazování v SQL

### testovací data
```sql
INSERT INTO Parrent (Name,Surname,Phone,Email,Address,Passwd) VALUES
('Jan', 'Pokorný', 123456789, 'j.pokorny@gmail.com', 'Ulice 34, Praha 6, 18000', 'safe_password'),
('Petr', 'Horký', 123456789, 'p.horky@gmail.com', 'JinaUlice 51, Praha 6, 18000', 'safe_password2');
INSERT INTO Child (ParrentID,Name,Surname,Phone,Email,Address,Birth,Alergies) VALUES
(2,'Pavel','Horký', NULL, NULL, 'JinaUlice 51, Praha 6, 18000', '10/04/1998', 'dust, soy'),
(2,'Tereza','Horká', NULL, NULL, 'JinaUlice 51, Praha 6, 18000', '10/04/1998', NULL),
(1,'Petra','Pokorná', NULL, NULL, 'Ulice 34, Praha 6, 18000', '01/09/2004', NULL);
INSERT INTO Event (Type,Name,Begins,Ends,Capacity) VALUES
('Víkendovka','JarVik','2020-03-23','2020-03-25',18),
('Víkendovka','PodVik','2020-10-02','2020-10-04',24),
('Tábor','LT2019','2019-07-24','2019-08-08',45),
('Tábor','LT2020','2020-07-23','2020-08-08',45);
INSERT INTO GoesTo (ChildID, EventID) VALUES
(1, 10),
(2, 10),
(3, 10),
(1, 13),
(2, 13),
(3, 13);
INSERT INTO Payment (ParrentID, Due, Date, Type) VALUES
(1, 4300, '2019-05-20', 'Transfer'),
(2, 4300, '2019-04-29', 'Card'),
(2, 4300, '2019-04-29', 'Card'),
(2, 4400, '2020-06-02', 'Card'),
(2, 4400, '2020-06-02', 'Card');
INSERT INTO PaidForWhat (PaymentID, EventID) VALUES
(1, 10),
(2, 10),
(3, 10),
(4, 13),
(5, 13);
INSERT INTO PaidForWhom (PaymentID, ChildID) VALUES
(1, 3),
(2, 1),
(3, 2),
(4, 1),
(5, 2);
```

### 1. dotaz
Vyber všechny děti Petra Horkého a setřiď je podle věku a pak podle jména.
```sql
SELECT Ch.Name, Ch.Surname, Ch.Birth 
FROM Child AS Ch, Parrent AS P
WHERE P.Name = 'Petr' and P.Surname = 'Horký' and P.ParrentID = Ch.ParrentID
ORDER BY Ch.Birth, Ch.Name
```

### 2. dotaz
Vyber všechny děti, které mají nějakou alergii.
```sql
SELECT Ch.Name, Ch.Surname
FROM Child as Ch
WHERE NOT EXISTS (
	SELECT *
	WHERE Ch.Alergies IS NULL
	)
```

### 3. dotaz
Vyber události s nejvyšší kapacitou pro účastníky.
```sql
SELECT E.Name
FROM Event AS E
WHERE E.Capacity >= ALL (
	SELECT E.Capacity
	FROM Event AS E
	)
```

### 4. dotaz
Vyber všechny děti, které jsou přihlášeny na 'LT2020' a nemají zaplaceno.
```sql
SELECT Ch.Name, Ch.Surname
FROM Child AS Ch, Event AS E, GoesTo AS G
	WHERE E.Name = 'LT2020' and G.ChildID = Ch.ChildID and G.EventID = E.EventID and Ch.ChildID NOT IN (
		SELECT Ch.ChildID
		FROM Child AS Ch, Event AS E, Payment AS P, PaidForWhat AS PFWT, PaidForWhom AS PFWM
		WHERE E.Name = 'LT2020' and PFWT.EventID = E.EventID and PFWT.PaymentID = P.PaymentID and
        PFWM.ChildID = Ch.ChildID and PFWM.PaymentID = P.PaymentID
		)
```


## Embedded SQL & DDL SQL

### Procedura
Procedura vloží novou instanci třídy ```Child```  včetně vazby na konkrétní instance tříd ```Parrent``` (1:N) a ```Event``` (M:N).

```sql
CREATE PROCEDURE createAndLogChild
@ParrentID int,
@EventName VARCHAR(50),
@Name VARCHAR(20),
@Surname VARCHAR(25),
@Phone int,
@Email VARCHAR(50),
@Address VARCHAR(50),
@Birth DATE,
@Alergies VARCHAR(128)
AS
BEGIN
	DECLARE @ParrentCheck int;
	DECLARE @EventCheck int;
	DECLARE @EventID int = 0;
	DECLARE @ChildID int;
	SELECT @ParrentCheck = COUNT(*) FROM Parrent AS P
		WHERE P.ParrentID = @ParrentID
	SELECT @EventID = E.EventID FROM Event AS E
		WHERE E.Name = @EventName
	IF @ParrentCheck != 1
		RAISERROR('Parrent with this ID does not exist.',1,1);
	ELSE
		BEGIN
			INSERT INTO Child (ParrentID,Name,Surname,Phone,Email,Address,Birth,Alergies) VALUES
			(@ParrentID,@Name,@Surname,@Phone,@Email,@Address,@Birth,@Alergies)
			SELECT @ChildID = Ch.ChildID FROM Child AS Ch
				WHERE Ch.Name = @Name and Ch.Surname = @Surname and Ch.Birth = @Birth
		END
	IF @EventID = 0
		RAISERROR('Event with this Name does not exist.',1,1);
	ELSE
		BEGIN
			INSERT INTO GoesTo (ChildID,EventID) VALUES
			(@ChildID,@EventID)
		END
END
```

### Funkce + trigger
Dítě nesmí být přihlášeno na jednu akci vícekrát a nemůže být přihlášeno, pokud je dosažena kapacita akce.

```sql
CREATE TRIGGER cannotLogIn ON GoesTo
AFTER INSERT
AS
	DECLARE @ChildID int;
	DECLARE @EventID int;
	DECLARE @DuplicityCheck int;
	DECLARE @CurrentlyLogged int;
	DECLARE @Capacity int;
	SELECT @ChildID = I.ChildID FROM inserted AS I
	SELECT @EventID = I.EventID FROM inserted AS I

	SELECT @DuplicityCheck = COUNT(*) FROM GoesTo AS G
		WHERE G.ChildId = @ChildID AND G.EventID = @EventID
	SELECT @CurrentlyLogged = COUNT(*) FROM GoesTo AS G
		WHERE G.EventID = @EventID
	SELECT @Capacity = E.Capacity FROM Event AS E
		WHERE E.EventID = @EventID
	IF @CurrentlyLogged > @Capacity
		BEGIN
			RAISERROR('Capacity of this event is full.',1,1)
			ROLLBACK TRANSACTION
		END
	IF @DuplicityCheck > 1
		BEGIN
			RAISERROR('This child is allready registered to the event.',1,1)
			ROLLBACK TRANSACTION
		END
```