Suppose Alice and Bob want to play a game of cards over the telephone. In this
game, each player has their own deck, which may be assembled from a large number
of legal cards. Alice and Bob don't trust each other, and each wants to make
sure the other can't surrepticiously cheat in any of the following ways:
* Drawing a card from anywhere other than the top of the deck.
* Creating cards out of thin air.
* Destroying cards that have been drawn from the deck.
Naturally, each wants to have these assurances while not having to divulge their
hand or the order of their deck to their rival.
The following protocol is proposed, which solves this problem.

When Alice assembles her deck:
1. She tells Bob which cards her deck uses.
2. She assigns each card a unique number. E.g. a number between 0 and 2^256.
   Note that no two cards in the deck should have the same number, even if
   they're interchangeable according to the rules of the game.
3. She creates for each of these assignments a zero-knowledge proof. The ZKP
   could be constructed such as

   zkp[i] = SHA256(card[i] + "=>" + nonce[i])

4. She tells Bob all of zkp.

When Alice shuffles her deck:
1.  She picks a key K1 for the entire deck. K1 should have the same bit length as
    the nonces.
2.  She transforms her deck into each card's corresponding nonce.
3.  She shuffles her deck.
4.  She encrypts each card with K1 using XOR encryption.
5.  She tells Bob her encrypted deck.
6.  Bob picks his own K2 using the same method in step #1. He shuffles Alice's
    deck and encrypts it using K2, again using XOR encryption.
7.  Bob tells Alice the double-encrypted deck.
8.  Alice decrypts the deck with her K1 key.
9.  She picks a new set of keys Ka[i], one for each card in the deck, and
    re-encrypts each card with its corresponding key.
10. She tells Bob her re-encrypted deck.
11. Bob decrypts the deck with his K2 key and re-encrypts each card with its
    Kb[i] key, then tells the doubly-re-encrypted deck to Alice.
Alice's deck is now shuffled in an order unknown to both.

When Alice draws a card from the deck onto her hand:
1. Alice asks Bob for the decryption key for the top card.
2. If Bob allows the action, he tells Alice his Kb[i] for that card.
3. Alice decrypts the card using Ka[i] ^ Kb[i] and adds it to her hand.
(Note: a "hand" is understood to mean those cards that Alice and only Alice has
seen after drawing it from the deck.)

When Alice plays a card:
1. Alice tells Bob the nonce of the card she's playing as well as the identity
   of the card itself (nonce[i] and card[i], as above).
2. Bob should validate that this matches the ZKP Alice gave before.
(Note: to "play" a card means to set it face up on the table so that its visible
to both players.)

After finishing, regardless of outcome:
1. Alice must reveal her entire hand.
2. Alice must draw and reveal all of remaining deck.
3. Bob should make consistency checks.
(Note: If the rules of the game demand that both players re assemble and
reshuffle their decks, for example to resolve a tie, this step should be
performed first.)

Other operations:

Alice draws a non-random card from the deck:
1. Alice asks for all remaining decryption keys from Bob.
2. Alice decrypts all cards, looks at all of them, picks one to add to her hand,
   and tells to Bob which one it is in encrypted form (encrypted by
   Ka[i]^Kb[i]).
3. The rest of the deck is reshuffled as done at the start of the game.
