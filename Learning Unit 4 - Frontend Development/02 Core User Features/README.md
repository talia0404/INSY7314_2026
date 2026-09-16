# 🎮 GameVault Frontend - Core User Feature

Now that registration and login are implemented, extend the frontend with the main GameVault user features:

* Browse games
* View game details
* Personal collections
* Reviews
* User profiles

The frontend should feel like **one connected application**, rather than five separate forms/pages.

The main navigation should provide:

```text
GameVault
-> Browse Games
-> My Collection
-> My Profile
-> Logout
```

Users should normally discover reviews through individual game details rather than having a separate Reviews item in the main navigation.

---

# 📁 1. Organise the frontend

Create components/pages for the new features. A suitable structure is:

```text
src
├── components
│   ├── GameCard.jsx
│   ├── ReviewForm.jsx
│   └── Navigation.jsx
├── pages
│   ├── GamesPage.jsx
│   ├── GameDetailsPage.jsx
│   ├── CollectionPage.jsx
│   └── ProfilePage.jsx
├── services
│   └── api.js
├── App.jsx
└── main.jsx
```

The responsibilities should remain separated:

```text
pages
-> complete screens

components
-> smaller reusable parts of screens

api.js
-> backend communication

App.jsx
-> application navigation and authentication
```

---

# 🧭 2. Add application navigation

Once a user is logged in, replace the simple "You are logged in" message with the main GameVault interface.

The navigation should contain:

```text
Browse Games
My Collection
My Profile
Logout
```

Use **React Router** so each major screen has its own route.

For example:

```text
/games
-> browse games

/games/:id
-> individual game details

/collection
-> logged-in user's collection

/profile
-> logged-in user's profile
```

Clicking a game should navigate to its details page rather than opening another form on the browse page.

The navigation flow should therefore feel like:

```text
Login
-> Browse Games
-> select a game
-> Game Details
-> add to collection or review game
```

Users should always be able to return to Browse Games using the main navigation.

---

# 🔎 3. Implement Browse Games

Create `GamesPage.jsx`.

When the page loads, request the available games from your backend using `api.js`.

The process should be:

```text
GamesPage loads
-> request GET /games
-> backend retrieves games from MongoDB
-> games returned
-> store games in React state
-> display GameCard for each game
```

Each `GameCard` should display enough information to help users choose a game, such as:

* title
* genre
* platform
* age rating
* release year
* availability

Each card must contain a **View Details** button.

Clicking it should navigate to:

```text
/games/:id
```

using that game's MongoDB `_id`.

Also provide loading, empty and error states. Do not display a blank screen while games are being retrieved.

---

# 🕹️ 4. Implement View Game Details

Create `GameDetailsPage.jsx`.

This page represents **one selected game**.

Retrieve the game ID from the URL and request:

```text
GET /games/:id
```

The page should display the complete information for that game.

It should also become the central location for actions relating to that game:

```text
Game Details
-> game information
-> Add to My Collection
-> existing reviews
-> Write a Review
```

This interaction is important. Users should not have to manually type a game ID into collection or review forms.

They select a game first, and the application already knows which game they are working with.

For example:

```text
Browse Games
-> select Minecraft
-> /games/minecraft-id
-> Game Details knows minecraft-id
-> Add to Collection automatically uses minecraft-id
```

Provide a way to return to Browse Games.

---

# ❤️ 5. Implement Personal Collections

Create `CollectionPage.jsx`.

A personal collection belongs to the **currently authenticated user**, so collection requests must send the JWT in the `Authorization` header.

Extend `api.js` with functions for the collection endpoints provided by your backend.

The main collection screen should retrieve and display the logged-in user's games:

```text
My Collection
-> authenticated request
-> backend identifies user from JWT
-> retrieve user's collection
-> display collected games
```

Each collected game should display its title and useful game information, plus a **Remove from Collection** action.

### Adding games

Do **not** create a form asking:

```text
Enter game ID: ______
```

Instead, the user should add a game from `GameDetailsPage`.

The button should know the current game's `_id` automatically:

```text
Game Details
-> Add to My Collection
-> current game ID sent to backend
-> collection updated
-> show success/error feedback
```

The collection page should then show that game when visited.

### Removing games

On `CollectionPage`:

```text
game in collection
-> Remove
-> send authenticated removal request
-> backend updates collection
-> frontend updates displayed collection
```

Ask for confirmation before removing an item if desired.

---

# ⭐ 6. Implement Reviews

Reviews should primarily exist within `GameDetailsPage`.

When viewing a game, retrieve reviews belonging to that game and display them underneath the game information.

Each review could show:

```text
user name
rating
review text
date
```

Create a reusable:

```text
ReviewForm.jsx
```

The form should contain:

**Rating:** use the range supported by your backend, for example 1–5.

**Review:** provide a text area where the user writes their review.

The user should **not** enter:

```text
game ID
user ID
user name
```

Those values should come from the application context:

```text
game ID
-> current GameDetailsPage

user
-> authenticated JWT/backend
```

Submitting should:

```text
ReviewForm
-> validate required fields
-> send JWT
-> send current game ID, rating and review
-> backend stores review
-> successful response
-> clear form
-> refresh/update displayed reviews
```

If the backend supports editing/deleting reviews, only display those actions on reviews belonging to the currently logged-in user.

---

# 👤 7. Implement User Profiles

Create `ProfilePage.jsx`.

When this page loads, use the JWT to request the existing protected profile endpoint:

```text
GET /auth/profile
```

The backend should identify the user from the JWT rather than requiring the user to enter their ID.

Display information such as:

```text
name
email
role
```

If your backend later supports profile editing, add an **Edit Profile** form here.

The edit form should begin with the user's **current information already filled in**. Users should not have to re-enter everything from scratch.

For example:

```text
Name
[Talia]

Email
[talia@example.com]
```

Submitting the form should:

```text
current profile
-> user edits information
-> submit
-> authenticated API request
-> backend validates changes
-> MongoDB updated
-> frontend displays updated profile
```

Do not allow a normal user to edit their `role` through this form.

---

# 🔐 8. Update `api.js` for authenticated requests

Your existing `api.js` should remain the central connection point.

Add functions for whichever endpoints your backend provides for:

```text
get all games
get one game
get collection
add to collection
remove from collection
get reviews
create review
get profile
update profile
```

For protected endpoints, retrieve:

```javascript
localStorage.getItem(
    "gamevaultToken"
)
```

and send:

```text
Authorization
-> Bearer <token>
```

You may create a small reusable helper inside `api.js` to avoid repeatedly constructing the same authentication header.

Do not send the JWT for endpoints that do not require authentication unless your design specifically needs it.

---

# 🔄 9. Make the features interact properly

The completed frontend should behave as one connected workflow:

```text
Register/Login
-> Browse Games
-> select game
-> View Game Details
-> add game to collection
-> write review
-> navigate to My Collection
-> selected game appears
-> navigate to My Profile
-> account information displayed
```

The key principle is that information already known by the application should **not be requested from the user again**.

For example:

```text
selected game
-> application already knows game ID

logged-in user
-> backend can identify user using JWT
```

Therefore, avoid forms asking users for technical values such as MongoDB IDs.

---

# 🧪 10. Test the complete frontend

Test the features as a connected user journey, not only individually.

Confirm that a user can:

1. Log in and browse all games.
2. Select a game and view its details.
3. Add the selected game to their personal collection.
4. View the game in My Collection.
5. Return to the game and submit a review.
6. See the new review without manually refreshing the entire application.
7. Open My Profile and see the correct logged-in user's information.
8. Remove a game from their collection.
9. Log out and lose access to authenticated screens.

Also test unsuccessful requests, missing games, empty collections, invalid reviews and expired/invalid authentication tokens.

## 📚 Resources

* [React — Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) — useful for understanding how the game, review and authentication components communicate.
* [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — useful when extending `api.js` with the new backend requests.
