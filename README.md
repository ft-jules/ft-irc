# ft_irc

Serveur IRC en C++98, projet du cursus 42 Paris, fait en binôme avec
Florence Cousergue ([@whoatebambi](https://github.com/whoatebambi)).

Un seul processus, des sockets non bloquantes, et une boucle d'événements qui
gère tous les clients en parallèle, sans thread.

## Compilation et lancement

```sh
make
./ircserv <port> <password>
```

Le port doit être entre 1024 et 65535.

## Se connecter

Avec irssi :

```sh
irssi -c 127.0.0.1 -p 6667 -n alice -w <password>
```

Ou à la main avec `nc`, chaque ligne terminée par `\r\n` :

```sh
nc -C 127.0.0.1 6667
PASS <password>
NICK alice
USER alice 0 * :Alice
JOIN #test
```

## Ce qui est géré

- authentification par mot de passe, puis `NICK` et `USER`
- canaux : `JOIN`, `PART`, `TOPIC`, `INVITE`, `KICK`
- messages : `PRIVMSG` vers un canal ou un utilisateur
- modes de canal : `i` (sur invitation), `t` (topic réservé aux opérateurs),
  `k` (clé), `l` (limite d'utilisateurs), `o` (donner ou retirer l'opérateur)
- `PING`, `QUIT`, et la négociation `CAP` qu'envoie irssi à la connexion

## Organisation

```
src/main.cpp          arguments et lancement
src/Server.*          socket d'écoute, boucle d'événements, liste des clients
src/Client.*          tampon de lecture, découpe des lignes, envoi aux commandes
src/Channel.*         membres, opérateurs, modes
src/Command*.*        une classe par commande IRC
src/Reply.*           réponses numériques du protocole
src/poll/             la boucle d'événements derrière une interface commune
```

`src/poll/` cache le mécanisme d'attente derrière une interface : `epoll` sous
Linux, `poll` ailleurs (macOS). Le serveur ne sait pas lequel il utilise.
