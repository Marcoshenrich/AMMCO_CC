Hi there, thank you for letting me take the AMMCO coding challenge. 

This was an interesting problem to solve because the optimal approach varies wildly depending on the circumstances in which the algorithm is called. 

I decided to use typescript for readability. I tried to optimize for simplicity and memory space, since friend networks can quickly become gargantuan if you are doing large querys. 

I started by declaring a User and a Movie class. The User class has a unique ID to help track visits during graph traversal, and the movie class has an ID for consistency. 

The User class contains arrays of friends and movies to appropriately traverse the network graph. In a more complex setting, I think making these fields objects with unique ID's as keys would be a more effective implementation. 


```typescript
export class User {
    id: string;
    friends: Array<User>;
    likedMovies: Array<Movie>;

    constructor(id: string, friends: Array<User>, likedMovies: Array<Movie>) {
        this.id = id;
        this.friends = friends;
        this.likedMovies = likedMovies;
    }
}

export class Movie {
    id: string;
    title: string;

    constructor(id: string, title: string) {
        this.id = id;
        this.title = title;
    }
}

```

Next, we build a function that accepts a user and a depth. The depth allows the dev to modulate how many degrees of separation they would like to define as a "network" for this function. 

We initialize an object to track unique movie titles and increment a count each time they are present in a user's movie list.

The Set and graph visitation logic is centered around reducing the total length of the queue at any time. A user is only added to the queue once. 

I decided to use a DFS approach with a declared stack because a large array is more mananageable on the memory side than crashing a recursive call stack when dealing with large networks. 

This type of pattern is typical with a BFS queue implementation, but Javascript's Array.shift() method is inefficient for large data sets. Therefore I used Array.pop() because it does not change the result of this particular algorithm, and is much faster. 

```typescript

type MovieTitle = string
type Count = number

interface FavoriteMovieObject {
    [key: MovieTitle]: Count
}

export const favoriteMovieInNetwork = (user: User, depth: number) => {
    if (depth < 0) return "Friend Network Depth Required"

    const movieLikesTracker = {} as FavoriteMovieObject;
    let userQueue: [User, number][] = [[user, depth]];
    const visitedUsers = new Set(user.id);
   
    while (userQueue.length) {

        let [currentUser, depth] = userQueue.pop()!;

        for (const movie of currentUser.likedMovies) {
            movieLikesTracker[movie.title] ||= 0;
            movieLikesTracker[movie.title]++;
        }

        depth--
        if (depth < 0) continue;

        for (const friend of currentUser.friends) {
            if (!visitedUsers.has(friend.id)) {
                visitedUsers.add(friend.id);
                userQueue.push([friend, depth]);
            }
        }
    }

    return findFavorite(movieLikesTracker)
}
```

Finally, a helper function is called to iterate through the movieLikesTracker and return the title with highest count seen. It does not account for ties, which would require additional logic and for the function to return an array or object. 


```typescript
const findFavorite = (movieLikesTracker: FavoriteMovieObject) => {
    let mostLikedMovie = "No Movies Liked In Network"
    let highestFound = -Infinity

    for (const [title, count] of Object.entries(movieLikesTracker)) {
        if (count > highestFound) {
            highestFound = count
            mostLikedMovie = title
        }
    }

    return mostLikedMovie
}

```

#Improvements / Extensions

Here are some ways that the algorithm could be improved and expanded on. 

1. Heap Logic
The N time complexity of iterating through the movieLikesTracker is generally acceptable for most use cases. However for large data sets, a Max Heap would be a much more efficient method of tracking the highest count seen. However, heaps are lengthy and complicated to implement in Javascript. 

2. Genericize the Traversal Algorithm
I considered building a version of the algorithm which could accept traversal and resolution callbacks. This would allow a single abstracted traversal algorithm to be used for various network traversal purposes. However, I chose a simpler more readable version for the purposes of the coding challenge. 

The Test

```javascript
test('favoriteMovieInNetwork finds the favorite movie in your network', () => {

    const alice = new User("1", [], [])
    const bob = new User("2", [], [])
    const joe = new User("3", [], [])
    const larry = new User("4", [], [])
    const tina = new User("5", [], [])

    const aliceFriends = [bob, joe]
    const bobFriends = [alice, larry, tina]
    const joeFriends = [alice]
    const larryFriends = [bob, tina]
    const tinaFriends = [bob, larry]

    const mov1 = new Movie("1", "Where the Wild Things Are")
    const mov2 = new Movie("2", "The Lion King")
    const mov3 = new Movie("3", "Brazil")
    const mov4 = new Movie("4", "Iron Man")
    const mov5 = new Movie("5", "Last of the Mohicans")
    const mov6 = new Movie("6", "Avengers")

    const aliceMovies = [mov1, mov4]
    const bobMovies = [mov1, mov2, mov6]
    const joeMovies = [mov3]
    const larryMovies = [mov2, mov5, mov6]
    const tinaMovies = [mov2]

    alice.likedMovies = aliceMovies
    bob.likedMovies = bobMovies
    joe.likedMovies = joeMovies
    larry.likedMovies = larryMovies
    tina.likedMovies = tinaMovies

    alice.friends = aliceFriends
    bob.friends = bobFriends
    joe.friends = joeFriends
    larry.friends = larryFriends
    tina.friends = tinaFriends


    expect(() => favoriteMovieInNetwork(alice, 3)).toBe("The Lion King");

    expect(() => favoriteMovieInNetwork(alice, 1)).toBe("Where the Wild Things Are");


});

```