# 🛡️ Step 03 — Admin Dashboard

The Admin Dashboard provides functionality that should only be available to users with the `admin` role.

Your backend already uses JWT authentication and role-based access control. The frontend should now use the authenticated user's role to determine whether the Admin Dashboard is available.

The main flow should be:

```text
admin logs in
-> frontend retrieves authenticated profile
-> role = admin
-> Admin Dashboard option displayed
-> admin opens dashboard
-> management tools displayed
```

A normal user should **not** see the Admin Dashboard option.

Importantly, hiding the dashboard is only a frontend usability measure. The backend's existing `authenticateToken` and `authorizeRoles("admin")` middleware must remain responsible for actually preventing unauthorised requests.

---

# 📁 1. Create the Admin Dashboard

Create a new page:

```text
src/pages/AdminDashboardPage.jsx
```

You may also create smaller components if required:

```text
src
├── components
│   └── admin
│       ├── GameForm.jsx
│       └── GameManagement.jsx
│
└── pages
    └── AdminDashboardPage.jsx
```

The dashboard should provide administrators with a central place to manage GameVault.

For the current backend, focus on **game management**:

```text
Admin Dashboard
-> view games
-> add game
-> edit game
-> delete game
```

Do not duplicate this functionality throughout the application. The dashboard should bring the administrative actions together.

---

# 🔐 2. Determine whether the user is an administrator

The frontend should use the authenticated user's profile information to determine their role.

Your protected profile endpoint can provide information such as:

```text
name
email
role
```

After authentication:

```text
JWT
-> GET /auth/profile
-> backend verifies JWT
-> user profile returned
-> frontend stores profile
-> check role
```

If:

```text
role = admin
```

display an **Admin Dashboard** option in the navigation.

If:

```text
role = user
```

do not display it.

Your authenticated navigation could therefore be:

```text
normal user
-> Browse Games
-> My Collection
-> My Profile
-> Logout
```

and:

```text
administrator
-> Browse Games
-> My Collection
-> My Profile
-> Admin Dashboard
-> Logout
```

---

# 🚫 3. Protect the dashboard route

Create a route such as:

```text
/admin
```

Do not rely only on hiding the navigation button.

If a normal user manually enters:

```text
http://localhost:5173/admin
```

the frontend should check their authenticated role and prevent the dashboard from being displayed.

The logic should be:

```text
/admin requested
-> check authenticated user
-> role = admin
-> display AdminDashboardPage
```

Otherwise:

```text
/admin requested
-> user missing or role is not admin
-> do not display dashboard
-> redirect to an appropriate page
```

Remember that this is still **frontend protection**. A user could potentially manipulate frontend JavaScript.

The real security remains:

```text
request reaches backend
-> authenticateToken
-> authorizeRoles("admin")
-> authorised request continues
```

---

# 🎮 4. Display the games management section

When `AdminDashboardPage` loads, retrieve the existing games.

Use the same game information already available through:

```text
GET /games
```

Display the games in a manageable format such as cards or a table.

Each game should show useful identifying information, for example:

```text
Title
Genre
Platform
Release Year
Age Rating
Available
```

For each game, provide:

```text
Edit
Delete
```

Also provide a clearly visible:

```text
Add New Game
```

button.

The dashboard should therefore look conceptually like:

```text
Admin Dashboard

Add New Game

Game 1
-> Edit
-> Delete

Game 2
-> Edit
-> Delete
```

---

# ➕ 5. Create the Add Game form

Create a reusable `GameForm` component.

When the administrator selects **Add New Game**, display an empty form.

The form must collect all information required by your backend's game creation validation:

```text
Title
Genre
Platform
Release Year
Age Rating
Available
```

Use appropriate controls.

For example:

**Title, Genre and Platform**

Use text inputs.

**Release Year**

Use a number input.

**Age Rating**

Prefer a dropdown containing the values accepted by your backend:

```text
E
E10+
T
M
18
```

Do not allow administrators to type arbitrary age ratings if the backend only accepts these values.

**Available**

Use a checkbox, switch or suitable selection control representing a Boolean value.

---

# 📤 6. Submit a new game

When the administrator submits the form:

```text
form submitted
-> prevent normal HTML submission
-> validate required fields
-> create game object
-> retrieve JWT
-> api.js
-> POST /games
-> backend authentication
-> backend admin authorisation
-> game validation
-> controller
-> MongoDB
-> response
```

The request must include:

```text
Authorization
-> Bearer <JWT>
```

because `POST /games` is an admin-protected endpoint.

Provide loading feedback while the request is being processed.

If successful:

```text
game created
-> clear/close form
-> update displayed games
-> show success feedback
```

The administrator should not need to manually refresh the browser to see the new game.

If unsuccessful, display the error returned by the backend.

---

# ✏️ 7. Implement Edit Game

When the administrator clicks **Edit** beside a game, reuse `GameForm` rather than creating an entirely separate form.

The important difference is that the form should now be populated with the selected game's existing information.

For example:

```text
administrator selects Edit for Minecraft
-> selected game passed to GameForm
-> Title already contains Minecraft
-> Genre contains existing genre
-> Platform contains existing platform
-> Release Year contains existing year
-> Age Rating contains existing rating
-> Available contains current value
```

The administrator then changes only what needs to be changed.

The form should clearly indicate that it is currently **editing** rather than adding a new game.

---

# 💾 8. Submit game changes

Your backend supports both:

```text
PUT /games/:id
PATCH /games/:id
```

Choose the endpoint based on what your form is designed to do.

If the edit form always sends **every game field**, use the full update operation.

If it sends only the values that changed, use the partial update operation.

Remember that your current backend validation treats these differently:

```text
PUT
-> all editable game fields required

PATCH
-> one or more editable fields required
```

The selected game's MongoDB `_id` should come from the game being edited. Do not ask the administrator to type an ID.

The request must also send the JWT.

After a successful update:

```text
backend updates game
-> updated game returned
-> close edit form
-> update games displayed on dashboard
-> administrator sees new information
```

---

# 🗑️ 9. Implement Delete Game

Each game should have a **Delete** option.

Do not immediately delete a game as soon as the button is clicked.

Ask the administrator to confirm the action.

For example:

```text
Delete "Minecraft"?

This action will remove the game from GameVault.

Cancel
Delete
```

If confirmed:

```text
selected game
-> retrieve its _id
-> retrieve JWT
-> DELETE /games/:id
-> backend authentication
-> admin authorisation
-> game removed from MongoDB
-> frontend updates game list
```

Again, the administrator should never have to enter the MongoDB ID manually.

If the deletion fails, keep the game displayed and show the backend error.

---

# 🌐 10. Extend `api.js`

Keep backend communication centralised in `api.js`.

Add functions for the administrative operations:

```text
create game
update game
delete game
```

You may already have a function for retrieving games from Step 02, so **reuse it** rather than creating an admin-specific version of the same GET request.

The API responsibilities should now include:

```text
getGames()
-> GET /games

createGame()
-> POST /games

updateGame()
-> PUT or PATCH /games/:id

deleteGame()
-> DELETE /games/:id
```

The create, update and delete requests must include the JWT.

Avoid writing separate `fetch()` requests directly inside every admin component.

---

# 🔄 11. Make the dashboard components work together

The page should control which game is currently being managed.

A useful interaction is:

```text
Admin Dashboard
-> Add New Game
-> empty GameForm displayed
-> submit
-> dashboard refreshes game list
```

For editing:

```text
Admin Dashboard
-> select existing game
-> Edit
-> selected game passed to GameForm
-> form populated
-> administrator makes changes
-> submit
-> dashboard updates
```

For deletion:

```text
Admin Dashboard
-> select Delete
-> confirmation
-> deletion request
-> game removed from dashboard
```

`GameForm` should therefore be reusable for both:

```text
creating
editing
```

rather than maintaining two nearly identical forms.

---

# ⚠️ 12. Handle loading, success and errors

The administrator should always understand what the application is doing.

Provide feedback for situations such as:

```text
loading games
creating game
updating game
deleting game
request successful
request unsuccessful
no games available
```

Disable relevant submission buttons while a request is being processed to reduce accidental duplicate requests.

Do not use browser console messages as the only feedback. Important information should be visible in the interface.

---

# 🧪 13. Test administrator access

Test with an account whose MongoDB role is:

```text
admin
```

Remember that if you manually change:

```text
user
-> admin
```

in MongoDB, log in again afterwards. The existing JWT may still contain the previous `user` role.

Confirm that an administrator can:

1. See the Admin Dashboard navigation option.
2. Open `/admin`.
3. View all games.
4. Add a valid game.
5. Edit an existing game.
6. Delete a game after confirmation.
7. See changes without manually refreshing the browser.

---

# 🔒 14. Test normal user access

This test is equally important.

Log in using:

```text
role = user
```

Confirm that:

```text
Admin Dashboard
-> not displayed in navigation
```

Then manually attempt to access:

```text
/admin
```

The frontend should prevent access.

Finally, remember that the backend must still reject administrative API requests from the normal user's JWT:

```text
normal user JWT
-> POST /games
-> authenticateToken
-> authorizeRoles("admin")
-> 403 Forbidden
```

This demonstrates the difference between **hiding functionality in the frontend** and **actually enforcing authorisation on the backend**.

# 📚 Resources

* [React — Conditional Rendering](https://react.dev/learn/conditional-rendering) — useful for displaying administrative controls according to the authenticated user's role.
* [React — Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) — useful when the dashboard and `GameForm` need to share the selected game and update the displayed data.
