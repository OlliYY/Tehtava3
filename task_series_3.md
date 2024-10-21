# Tehtäväsarja 3 - Keskustelupalstan toteutus

käytin tämän tehtävän tekemiseen supbasea.

## 1. Tietokantataulun luominen

Loin `posts`-taulun käyttäen SQL-Editoria seuraavilla kentillä:
- `id`: Pääavain, automaattisesti kasvava kokonaisluku (is identity).
- `content`: Viestin sisältö.
- `created_at`: Aikaleima, jolloin viesti luotiin.
- `parent_post_id`: Viittaa toiseen viestiin (itsereferenssi).
- `user_id`: Viittaa käyttäjään, joka loi viestin, voi olla `NULL` anonyymiviesteille.
- `title`: Viestin otsikko tai aihe.

kuva taulun luonnista
![alt text](<Näyttökuva 2024-10-21 220238.png>)
## 2. Testidatan lisääminen

Lisäsin aloituspostauksen ja vastauksen testatakseni toimintaa.

- **Aloitusviestin lisääminen**:
  ```sql
  INSERT INTO posts (content, user_id, title) 
  VALUES ('Tämä on aloitusviesti', '6fcb8b18-0b9a-4583-8117-51fa2a8d040d', 'Ensimmäinen aihe');
  ```

kuva aloitusviestin lähettämisestä
![alt text](<Näyttökuva 2024-10-21 220406.png>)

- **Vastausviestin lisääminen**:
  ```sql
  INSERT INTO posts (content, parent_post_id, user_id) 
  VALUES ('Tämä on vastaus aloitusviestiin', 1, NULL);
  ```

kuva vastausviestistä
![alt text](<Näyttökuva 2024-10-21 220455.png>)



## 3. Datan hakeminen

Toteutin seuraavat kyselyt, joilla haettiin aloitusviestit ja niiden vastaukset:

- **Aloitusviestien hakeminen**:
  ```sql
  SELECT * FROM posts WHERE parent_post_id IS NULL;
  ```

kuva aloitusviestien hakemisesta
![alt text](<Näyttökuva 2024-10-21 220644.png>)

- **Vastausten hakeminen tiettyyn aloitusviestiin**:
  ```sql
  SELECT * FROM posts WHERE parent_post_id = 1;
  ```

kuva vastausviestien hakemisesta
![alt text](<Näyttökuva 2024-10-21 220711.png>)

## 4. Viestien poistaminen

Testasin viestin poistamista ja tarkistin, että siihen liittyvät vastaukset poistettiin automaattisesti.
 Käytin poistamiseen ON DELETE CASCADE-ehtoa joka poistaa
kaikki vastaukset, kun aloitusviesti poistetaan

- **Viestin poistaminen**:
  ```sql
  DELETE FROM posts WHERE id = 1;
  ```

kuva viestin poistamisesta
![alt text](<Näyttökuva 2024-10-21 220752.png>)

## 5. Tykkäystoiminnon lisääminen

Loin `likes`-taulun, joka mahdollistaa tykkäyksien lisäämisen viesteihin.Käyttäjä voi tykätä viestistä vain kerran, 
mikä estetään tekemällä likes-taulun post_id ja user_id yhdistelmästä pääavain (PRIMARY KEY). Jos käyttäjä yrittää tykätä uudelleen,
 tietokanta estää sen. likes-taulu viittaa posts-tauluun ja auth.users-tauluun, mikä varmistaa tietokannan eheyden.

```sql
CREATE TABLE likes (
  post_id INT REFERENCES posts(id),
  user_id UUID REFERENCES auth.users(id),
  PRIMARY KEY (post_id, user_id)
);
```
kuva like tablen luonnista
![alt text](<Näyttökuva 2024-10-21 220911.png>)

Lisäsin testitykkäyksen seuraavasti:

```sql
INSERT INTO likes (post_id, user_id) 
VALUES (4, '6fcb8b18-0b9a-4583-8117-51fa2a8d040d');
```

kuva tykkäyksen lisäämisestä
![alt text](<Näyttökuva 2024-10-21 221257.png>)

Tykkäysten laskeminen:

```sql
SELECT post_id, COUNT(*) AS like_count 
FROM likes 
GROUP BY post_id;
```

kuva tykkäyksistä
![alt text](<Näyttökuva 2024-10-21 221509.png>)
