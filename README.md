# Movie

Create database films;
use films;


-- Създаване на таблицата за студио
CREATE TABLE Studio (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    bulstat VARCHAR(20) NOT NULL
);

-- Създаване на таблицата за продуцент
CREATE TABLE Producer (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    bulstat VARCHAR(20) NOT NULL
);


CREATE TABLE Movie (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    release_year INT NOT NULL,
    length INT NOT NULL,
    budget DECIMAL(15, 2) NOT NULL,
    studio_id INT,
    producer_id INT,
    CONSTRAINT fk_studio
        FOREIGN KEY (studio_id) REFERENCES Studio(id),
    CONSTRAINT fk_producer
        FOREIGN KEY (producer_id) REFERENCES Producer(id)
);

-- Създаване на таблицата за актьор
CREATE TABLE Actor (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    gender ENUM('Male', 'Female') NOT NULL,
    birth_date DATE NOT NULL
);

-- Създаване на таблицата за участие (casting)
CREATE TABLE Casting (
    movie_id INT,
    actor_id INT,
    role VARCHAR(255),
    PRIMARY KEY (movie_id, actor_id),
    CONSTRAINT fk_movie
        FOREIGN KEY (movie_id) REFERENCES Movie(id),
    CONSTRAINT fk_actor
    foreign key (actor_id) references Actor(id)
    );
    
    -- Вмъкване на данни в таблицата за студиа
INSERT INTO Studio (name, address, bulstat) VALUES 
('Warner Bros', 'Burbank, California, USA', '123456789'),
('Universal Pictures', 'Universal City, California, USA', '987654321');

-- Вмъкване на данни в таблицата за продуценти
INSERT INTO Producer (name, address, bulstat) VALUES 
('Steven Spielberg', 'Los Angeles, California, USA', '111222333'),
('Christopher Nolan', 'London, England, UK', '444555666');

-- Вмъкване на данни в таблицата за актьори
INSERT INTO Actor (name, address, gender, birth_date) VALUES 
('Leonardo DiCaprio', 'Los Angeles, California, USA', 'Male', '1974-11-11'),
('Morgan Freeman', 'Charleston, Mississippi, USA', 'Male', '1937-06-01'),
('Scarlett Johansson', 'New York City, New York, USA', 'Female', '1984-11-22'),
('Tom Hanks', 'Concord, California, USA', 'Male', '1956-07-09');

-- Вмъкване на данни в таблицата за филми
INSERT INTO Movie (title, release_year, length, budget, studio_id, producer_id) VALUES 
('Inception', 2010, 148, 160000000, 1, 2),
('Jurassic Park', 1993, 127, 63000000, 2, 1);

-- Вмъкване на данни в таблицата за участие (casting)
INSERT INTO Casting (movie_id, actor_id, role) VALUES 
(1, 1, 'Dom Cobb'),
(1, 3, 'Mal Cobb'),
(2, 2, 'Dr. John Hammond'),
(2, 4, 'Dr. Alan Grant');


/* заявка, която извежда имената на всички актьори, които са мъже */
select  name
from Actor
where gender ='Male';

/*заявка, която извежда топ 3 на най-високо бюджетните филми, излезнали между 1990 и 2000 година.*/
select title, release_year,budget
from Movie
WHERE release_year BETWEEN 1990 AND 2000
ORDER BY budget DESC
LIMIT 3;
	
    /*заявка, която извежда имена на филми и имената на актьорите, които участват в тях, но само за филми, които са правени от студио „IFS-200”.*/
SELECT Movie.title AS movie_title, Actor.name AS actor_name
FROM Movie
JOIN Casting ON Movie.id = Casting.movie_id
JOIN Actor ON Casting.actor_id = Actor.id
JOIN Studio ON Movie.studio_id = Studio.id
WHERE Studio.name = 'IFS-200';


/* заявка, която извежда списък на всички филми, заедно с името на студиото, в което са правени и името на продуцента им.*/
SELECT Movie.title AS movie_title, Studio.name AS studio_name, Producer.name AS producer_name
FROM Movie
JOIN Studio ON Movie.studio_id = Studio.id
JOIN Producer ON Movie.producer_id = Producer.id;

/* заявка, която извежда имената на актрисите, играли във филма „MGM“.*/
SELECT DISTINCT a.name   /*Избира уникалните имена на актрисите.*/
FROM Actor a
JOIN Casting c ON a.id = c.actor_id
JOIN Movie m ON c.movie_id = m.id
JOIN Studio s ON m.studio_id = s.id
WHERE m.title = 'MGM' AND a.gender = 'Female';

/*заявка, която извежда името на актьора, който е играл в най-малко филми.*/
SELECT a.name AS actor_name
FROM Actor a
JOIN Casting c ON a.id = c.actor_id
GROUP BY a.id, a.name
ORDER BY COUNT(c.movie_id) ASC
LIMIT 1;

/*заявка, която извежда най – стария от мъжете актьори, снимал се в най-много филми.*/
SELECT Actor.name, Actor.birth_date, COUNT(Casting.movie_id) AS num_movies
FROM Actor
JOIN Casting ON Actor.id = Casting.actor_id
WHERE Actor.gender = 'Male'
GROUP BY Actor.id, Actor.name, Actor.birth_date
ORDER BY COUNT(Casting.movie_id) DESC, Actor.birth_date ASC
LIMIT 1;

/* заявка, която извежда имената на актьорите, които не са участвали в нито един филм.*/
SELECT name
FROM Actor
WHERE id NOT IN (
    SELECT DISTINCT actor_id
    FROM Casting
);
