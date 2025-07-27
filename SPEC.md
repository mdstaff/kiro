An Iterative Learning Path: Building a Scalable Multiplayer Card Game with Elixir/Phoenix
Introduction
Welcome to Elixir and Phoenix, a powerful combination that offers a distinct approach to building highly concurrent, distributed, and fault-tolerant applications. This report outlines a structured, iterative plan for experienced JavaScript/TypeScript developers to master this ecosystem by constructing a local web-based multiplayer card game platform. The journey will highlight how Elixir and Phoenix are particularly well-suited for real-time systems, such as multiplayer games and chat servers, due to their foundational design principles.
Elixir, built upon the robust Erlang Virtual Machine (BEAM), provides inherent capabilities for developing systems that demand high availability and resilience.1 Its functional programming paradigm inherently promotes immutability and declarative code, contributing to highly predictable and maintainable systems.1 Phoenix, as the leading web framework in the Elixir ecosystem, fully leverages these strengths. It offers powerful, built-in tools like Channels for real-time communication and LiveView for constructing rich, interactive user interfaces with significantly reduced client-side JavaScript.4 This integrated approach simplifies the development of complex real-time features that would typically require extensive client-side JavaScript frameworks and intricate state synchronization.
For developers accustomed to JavaScript/TypeScript, transitioning to Elixir involves a significant shift from familiar object-oriented and imperative programming styles to functional programming and the actor model. A core difference is Elixir's strict emphasis on immutability: data structures cannot be changed after creation; instead, operations return new, modified copies.1 This contrasts sharply with the mutable objects and stateful components prevalent in JS/TS. The
actor model, central to Elixir via the BEAM VM, introduces "processes" as isolated, lightweight units of concurrency that communicate exclusively through asynchronous message passing, rather than shared memory or traditional threads.1 This architectural approach inherently promotes fault tolerance and scalability, representing a fundamental departure from typical threading models found in other languages.
A common challenge in traditional imperative/object-oriented models is the management of errors and unexpected failures. Developers often write extensive defensive code to prevent every possible error. In contrast, Elixir, leveraging the BEAM, embraces a philosophy often referred to as "let it crash".7 This perspective arises from the inherent isolation of Elixir processes.2 When one process encounters an error and terminates, it does so in isolation without affecting other parts of the system. The system is designed to automatically restart the faulty process to a known good state, often under the guidance of a supervisor (a concept explored in Phase 4). This approach simplifies error handling by reducing the need for complex recovery logic within every function. More importantly, it directly enables high availability and scalability, as the system as a whole remains operational and resilient even under stress or unexpected conditions. This represents a fundamental shift in how one approaches system design for robustness and concurrency.
The actor model, where isolated processes communicate via message passing, is a core tenet of Elixir's concurrency.1 This design has profound practical implications for building a multiplayer game. In traditional development, a single game server might manage all game states and client connections, which can become a bottleneck and a single point of failure. With the actor model, each active game instance, or even each player's individual connection, can be represented by its own isolated Elixir process.9 If a specific game process encounters an error (e.g., due to a bug in complex game logic), only that single process terminates. The system's supervision strategy (e.g., "one-for-one," to be discussed in Phase 4) ensures that only the faulty game instance is restarted, leaving all other active games and the overall platform unaffected. This architecture inherently supports horizontal scaling and resilience, allowing the platform to handle a large number of concurrent games and players with high availability. This is a powerful, built-in advantage for real-time multiplayer applications, simplifying the challenges of distributed systems.
This learning plan is designed as a hands-on journey to build a web-based multiplayer card game platform, encompassing games like Hearts, Spades, Sheepshead, Solitaire, and Golf. The primary goal is not merely to construct a game, but to gain a deep, practical understanding of Elixir and Phoenix's unique strengths, particularly in handling real-time interactions, managing concurrent game states, and ensuring system resilience. The diverse game types will provide varied and engaging challenges for implementing game logic and managing complex state.
Throughout this journey, AI/LLMs can serve as invaluable learning companions. These tools can be utilized for quick syntax lookups, clarifying complex functional programming concepts, generating boilerplate code, assisting with debugging, and exploring alternative implementations. For example, a developer might ask an LLM to draw analogies between Elixir's Map.update! and JavaScript's Object.assign or spread syntax to connect new concepts to existing knowledge. However, it is important to remember that AI is a tool; critical thinking, hands-on coding practice, and deep conceptual understanding remain paramount for achieving true mastery.

Phase 1: Foundations – Getting Started with Elixir & Phoenix


Setting Up the Elixir/Phoenix Development Environment

Establishing a robust development environment is the initial and critical step in any new programming journey. For Elixir and Phoenix, the most recommended and flexible tool for managing language versions is asdf.10 This version manager simplifies the installation and management of various programming languages, offering a unified way to install and switch between different runtimes. This capability is invaluable for managing dependencies across various projects and ensuring compatibility without manual intervention.10
The installation process with asdf typically involves a few straightforward steps:
The first action is to install asdf itself. Detailed instructions specific to the operating system (Linux, macOS, Windows) are available in the official asdf documentation.10
Once asdf is installed, the Elixir plugin must be added to enable asdf to manage Elixir versions. This is achieved by running the command: $ asdf plugin add elixir.10
Next, Erlang, a fundamental prerequisite for running Elixir, needs to be installed. It is crucial to install the correct Erlang/OTP version that is compatible with the chosen Elixir version. asdf facilitates this by allowing installation of the latest stable version or a specific one: $ asdf install erlang <version>.10
Finally, Elixir itself can be installed: $ asdf install elixir <version>.10
To make these versions the default for the system or a specific project, commands like $ asdf global elixir <version> erlang <version> are used.
While asdf is highly recommended for its flexibility and ease of version management, several alternative installation methods are available depending on the operating system. These include OS-specific package managers such as Homebrew (brew install elixir on macOS), pacman (Arch Linux), dnf (Fedora), or apt (Ubuntu, often via the RabbitMQ PPA).11 For Windows users, platform-specific installers or tools like Scoop and Chocolatey provide convenient options.11 For quick experimentation or isolated environments, Docker offers a viable alternative, allowing developers to get started quickly with a pre-configured Elixir image by running
docker run -it --rm elixir.11
After installation, it is advisable to verify the setup by launching the Erlang shell (erl) and the Elixir interactive shell (iex).10 Ensuring the system's
PATH environment variable is correctly configured to include Elixir's binary path is also important for seamless development.11
To initiate the project, the Phoenix project generator is utilized: $ mix phx.new my_card_game --live --no-html --no-ecto. The --live flag incorporates Phoenix LiveView for interactive user interfaces, while --no-html and --no-ecto streamline the initial setup by omitting traditional HTML views and database configuration, as these components will be introduced iteratively as the project progresses.

Understanding the Phoenix Project Structure

Phoenix's project structure is meticulously designed to promote a clear separation of concerns, particularly distinguishing the web presentation layer from the core business logic.12 This modularity is a key factor in maintaining application clarity, scalability, and long-term viability.
Key directories within a Phoenix project and their respective purposes include:
_build/: This directory contains compiled application files tailored for different environments, such as development and production.4
.elixir_ls/: This folder stores files utilized by the Elixir Language Server (ElixirLS), which provides rich Integrated Development Environment (IDE) features like code completion, diagnostics, and inline documentation.4
assets/: Frontend-related files reside here, encompassing CSS (often configured with Tailwind CSS for styling) and JavaScript files (e.g., for AlpineJS interactivity).4
config/: This directory holds configuration files specifically designed for various environments, including dev.exs for development, prod.exs for production, test.exs for testing, and runtime.exs for settings evaluated at runtime, useful for secrets.4
deps/: External dependencies fetched by the mix deps.get command are stored within this folder.4
lib/: This directory represents the core of the backend application. It is typically subdivided into your_app/ (housing core business logic, contexts, and Ecto schemas) and your_app_web/ (containing the web interface components, such as controllers, views, LiveViews, templates, and Channels).4 This explicit separation of web and business logic layers is a notable difference from frameworks like Ruby on Rails, where the
app/ directory might blend these concerns.12
priv/: This folder is used for static files, Gettext translation files, and Ecto migration files, which manage database schema changes.4
test/: All unit and integration test cases for the application are organized within this directory.4
Beyond these directories, mix.exs and mix.lock are crucial files for managing the project's dependencies and configuration settings. mix.lock specifically ensures repeatable builds by locking specific dependency versions, which is vital for consistent development and deployment environments.4
The "batteries included" philosophy in Elixir and Phoenix, while offering high productivity, is fundamentally different from many other frameworks. Their solutions, such as LiveView for interactive UIs and Channels for real-time communication, are inherently designed for concurrency, distribution, and fault tolerance because they are built upon functional programming principles and the actor model. This means that developers gain a significantly more robust and scalable application from the ground up, even if the initial learning curve for the paradigm shift is steeper than learning a new JavaScript framework. This foundational strength allows developers to concentrate on core application logic rather than grappling with complex real-time infrastructure or manual distributed state management, leading to a more efficient development process for highly interactive applications.

Elixir Basics: Syntax, Data Types, Pattern Matching, Modules, Functions

Elixir is a dynamically typed functional programming language engineered for building stable and maintainable applications.13 Its design emphasizes clarity and robustness, drawing heavily from the Erlang VM's capabilities.
Modules and Functions: Code organization in Elixir revolves around modules, which serve as containers for named functions.3 Functions in Elixir are first-class citizens, meaning they can be passed as arguments, returned from other functions, and assigned to variables, a characteristic that supports flexible and composable code.2
Variables: In Elixir, the = operator is primarily a match operator, not an assignment in the imperative sense. Variables are immutable; when a variable appears to be "updated," a new value is actually created, and the variable name is rebound to this new value.13 This immutability is a cornerstone of functional programming, contributing to predictable behavior and simplifying concurrency.
Data Types: Elixir provides a rich set of built-in data structures:
Lists: These are ordered collections of elements, commonly used for sequences of data, such as representing a deck of cards.3
Maps: Flexible key-value data structures, similar to JavaScript objects, where keys can be of any type (e.g., atoms, strings, integers).3 They are versatile for storing unstructured or dynamic data.
Structs: Built on top of maps, structs provide compile-time checks and allow for default values.3 Defined using
defstruct, they enforce a predefined shape for data, preventing the addition of undefined keys and ensuring data consistency.15 Structs are highly recommended for representing well-defined entities like a
Card, Player, or Game state, as they bring a level of type safety and clarity to the data model.3
Pattern Matching: This is one of Elixir's most powerful features, enabling concise and readable code, particularly for defining game rules and handling different scenarios.2 It allows developers to match on values, data structures, and even function arguments, leading to elegant and expressive conditional logic.3 This capability simplifies complex branching and makes code easier to understand and maintain.

Your First Phoenix LiveView: Interactive UI without JavaScript

Phoenix LiveView is a transformative framework that enables the creation of highly interactive, real-time web applications by pushing HTML updates over WebSockets, significantly reducing the need for extensive client-side JavaScript.4 This approach fundamentally changes how web applications are built, centralizing UI logic and state management on the server.
The initial page render in LiveView is a static HTML response, which is advantageous for Search Engine Optimization (SEO) and initial page load performance.5 Subsequent dynamic updates and interactions are handled over a persistent, bidirectional WebSocket connection, ensuring a fluid user experience without full page reloads.16
A fundamental LiveView module typically implements two core callbacks:
render/1: This function defines how the component is represented in HTML. It interpolates elements of the component's state, known as assigns, using the ~L sigil.5 This is akin to a templating engine that reacts to state changes.
mount/2: This callback executes at the beginning of the LiveView's lifecycle. Its primary responsibility is to set up the initial state of the LiveView and wire up any necessary socket assigns.5
User interactions, such as button clicks or form submissions, are managed by handle_event/3 callbacks. These callbacks process the incoming events, mutate the LiveView's state, and automatically trigger an efficient re-render of the affected parts of the UI to reflect the changes.5
To make a LiveView accessible via a web browser, it is wired up in the router.ex file using the live "/path", YourLiveView macro.5 On the client-side, the
phoenix_live_view.js library handles the underlying WebSocket connection, manages events, and efficiently patches the Document Object Model (DOM) to reflect server-side changes, minimizing network traffic and client-side processing.16
LiveView's promise of building "interactive, real-time applications without requiring extensive JavaScript" 4 is a compelling statement for JS/TS developers. The core mechanism involves LiveView centralizing UI state management and rendering logic on the server. Instead of a complex client-side framework (like React or Vue) managing the entire DOM, LiveView sends minimal HTML differences over WebSockets. This means that intricate logic for managing elements like card positions, player hands, game board updates, and turn indicators, which would typically involve significant client-side state, is primarily handled by server-side Elixir code. This approach significantly reduces the need for a separate, complex frontend framework and the associated overhead of designing REST APIs, managing client-server data synchronization, and handling complex client-side state. While some client-side JavaScript might still be necessary for highly specific animations or integrations (e.g., WebGL for custom card rendering), the core reactivity and UI updates are server-driven. This simplifies the overall application architecture, allowing the JS/TS developer to leverage their existing UI/UX understanding while focusing on the powerful Elixir backend.
Table 1: Elixir/Phoenix Installation Methods (OS-Specific)
Operating System
Recommended Method(s)
Key Commands/Notes
macOS
asdf, Homebrew, Macports, Install Scripts
$ asdf plugin add elixir, $ asdf install elixir <version> erlang <version>, brew install elixir, sudo port install elixir, curl -fsSO https://elixir-lang.org/install.sh sh install.sh elixir@1.18.4 otp@27.3.4 10
Linux (Ubuntu)
asdf, Install Scripts, RabbitMQ PPA
$ asdf plugin add elixir, $ asdf install elixir <version> erlang <version>, curl -fsSO https://elixir-lang.org/install.sh sh install.sh elixir@1.18.4 otp@27.3.4, sudo add-apt-repository ppa:rabbitmq/rabbitmq-erlang, sudo apt update, sudo apt install git elixir erlang 10
Linux (Arch, Fedora, Gentoo, GNU Guix)
Package Managers
pacman -S elixir (Arch), sudo dnf install elixir erlang (Fedora), emerge --ask dev-lang/elixir (Gentoo), guix package -i elixir (GNU Guix) 11
Windows
asdf, Installers, Scoop, Chocolatey, Install Scripts
$ asdf plugin add elixir, $ asdf install elixir <version> erlang <version>, Download Erlang/Elixir installers, scoop install erlang elixir, choco install elixir, curl.exe -fsSO https://elixir-lang.org/install.bat.\install.bat elixir@1.18.4 otp@27.3.4 10
Docker
Official Docker Image
docker run -it --rm elixir (interactive shell), docker run -it --rm elixir bash (bash within container) 11

Note: Always ensure Erlang/OTP version compatibility with your chosen Elixir version. mix phx.new my_app --live --no-html --no-ecto is the command to create a new Phoenix project for this plan.

Phase 2: Core Game Logic – Functional Design


Designing Card Game Data Structures using Elixir Structs and Maps

Elixir's functional nature, combined with its rich set of immutable data structures—lists, maps, and structs—provides an excellent foundation for representing complex game states clearly and efficiently.3 This approach inherently supports the development of robust and predictable game logic.
A Card can be elegantly represented as an Elixir struct, which provides compile-time guarantees for its attributes. For instance, defstruct [:rank, :suit] could define a Card struct, with instances appearing as %Card{rank: :ace, suit: :spades}.15 This design ensures that all cards conform to a consistent shape and prevents the accidental addition of undefined fields, which enhances code reliability.15
A Deck of cards is naturally modeled as an Elixir List of Card structs.18 Similarly, a player's
Hand can also be a List of cards. If specific card lookups by an identifier are frequently needed within a hand, a Map could also be considered, though a list is often sufficient for ordered collections. The CardDeck library 18 provides a good example of how to define
card(), deck(), rank(), and suit() types, illustrating best practices for card representation.
A Player can be another struct, containing essential information such as their name, id, current hand (a list of cards), score, and status (e.g., :active, :folded).3 Structs ensure that player data is consistently structured throughout the application.
The overall Game state for a single match should be encapsulated within a single, immutable struct.3 This
Game struct would hold all dynamic information relevant to the current game: a list or map of Player structs, the deck, discard_pile, current_turn, game_phase (e.g., :dealing, :playing_trick, :scoring), and any other game-specific variables.3 This monolithic game state struct ensures internal consistency and simplifies the process of passing the state between functions, making the game logic easier to reason about.
A core principle in Elixir is immutability. When a game move or event occurs, the existing Game struct is not modified in place. Instead, a new Game struct is returned with the updated state.3 This functional approach significantly simplifies debugging, makes reasoning about state changes straightforward, and inherently avoids side effects, which is crucial for concurrent systems. This design choice contributes to more predictable and maintainable code.3 Developers can reason about the flow of data with greater confidence, as the state at any given point is a snapshot, not a moving target. This significantly reduces the cognitive load of managing concurrent interactions and contributes directly to the overall stability and reliability of the game platform.

Implementing Core Game Mechanics with Pure Functions and Pattern Matching

Elixir's functional programming paradigm, with its emphasis on pure functions and powerful pattern matching, is exceptionally well-suited for implementing the clear, rule-based logic inherent in card games.
Core mechanics such as shuffling and dealing cards are implemented using pure functions. Libraries like CardDeck offer functions such as CardDeck.shuffle/1 and CardDeck.deal/2.18 For instance, dealing cards to multiple players can involve splitting a list representing the deck.19 These operations are designed to take an input (e.g., the current deck) and return a new output (e.g., new decks and hands) without altering the original data. This characteristic of pure functions makes them inherently predictable and easy to test.
Elixir's pattern matching is exceptionally powerful for defining complex game rules in a declarative and concise manner.2 For example, a
Rules module could feature multiple function clauses that pattern match on different game states or card properties. The Rules.move_spaces/3 example in a board game context demonstrates how pattern matching can elegantly handle specific conditions, such as preventing a jailed player from moving.3 This principle is directly applicable to card game rules; for instance, a function
handle_play_card(%Game{phase: :playing_trick, current_suit: :spades}, %Card{suit: :spades} = card) could enforce the rule of "following suit." This approach allows each game rule (e.g., "what happens when an Ace is played," "how to handle a tie in War") to be expressed as a distinct function clause or a small, focused function. This creates a highly declarative codebase where the code directly reflects the game's rules and logic, rather than being obscured by imperative control flow. This leads to highly readable, maintainable, and extensible game logic. Adding new games or modifying existing rules becomes a matter of adding or changing specific function clauses, rather than navigating tangled imperative code. This directly impacts the long-term viability and ease of expansion for the multiplayer card game platform.
The entire game logic can be effectively modeled as a recursive function that operates on the immutable game state.20 Each "turn," "action," or "phase transition" is represented by a function that takes the current
Game state as input and returns a new, updated Game state.3 This creates a clear, auditable flow of state changes, akin to a state machine where each function represents a permissible transition.
Elixir's built-in testing framework, ExUnit, simplifies writing and running tests for game logic.3 The use of pure functions, which have no side effects and predictable outputs, makes them inherently easier to test in isolation, ensuring the accuracy and reliability of game mechanics.
Table 2: Core Elixir Data Structures for Card Games
Elixir Data Structure
Common JS/TS Analogue
Card Game Application
Example Syntax/Usage
Key Benefits/Characteristics
Map
JavaScript Object ({})
Flexible storage for dynamic data, e.g., player preferences, temporary game settings.
%{key: "value", "another_key" => 123} 14
Flexible key-value store, dynamic keys, pattern matching 14
Struct
Class Instance / Interface
Well-defined entities like Card, Player, Game state. Ensures data shape.
%Card{rank: :ace, suit: :spades}, %Player{name: "Alice", hand:} 3
Compile-time checks, default values, enforces data shape, immutability 15
List
JavaScript Array (``)
Ordered collections like a Deck of cards, a Hand, a Discard Pile.
[card1, card2, card3], Enum.random(1..100) 13
Ordered collection, efficient for head/tail operations, immutability 3


Phase 3: Real-time Multiplayer – Phoenix Channels & LiveView


Introduction to the BEAM VM, Processes, Concurrency, and Fault Tolerance

The Erlang Virtual Machine (BEAM) serves as the robust foundation upon which Elixir operates. It functions as a single operating system process and is responsible for the creation, scheduling, and management of lightweight Erlang/Elixir processes.1 BEAM is intrinsically designed for low-latency, real-time processing, rendering it exceptionally well-suited for applications with stringent timing requirements, such as multiplayer games.1
A key distinction of BEAM processes is their nature: unlike traditional operating system threads, they are incredibly lightweight, isolated, and communicate exclusively via asynchronous message passing, rather than shared memory.1 This fundamental isolation is the cornerstone of Elixir's fault tolerance; if an error occurs within one process, it terminates in isolation without affecting other processes or the overall system.2 This isolation is a powerful design choice that simplifies error handling and contributes to system stability.
BEAM's sophisticated schedulers efficiently distribute these millions of lightweight processes across available CPU cores, enabling the creation of highly scalable and parallel applications.2 Real-world examples, such as WhatsApp handling billions of messages daily, and large-scale IoT deployments managing millions of concurrent connections, demonstrate BEAM's proven capability to manage massive concurrent interactions and data volumes with minimal downtime.1
Elixir, by leveraging the BEAM, provides powerful, built-in constructs from the Open Telecom Platform (OTP) framework, such as supervisors and monitors, to handle failures gracefully.1 This embodies the "let it crash" philosophy 8, ensuring system reliability and continuous operation even in the presence of errors, which is critical for a robust multiplayer game platform. This means that instead of attempting to prevent every conceivable failure, the system is designed to expect and gracefully recover from crashes, automatically restarting faulty components to a known good state.

Phoenix Channels: The Backbone of Real-time Communication

Phoenix Channels offer a powerful and elegant abstraction for adding soft-realtime features to an application, built upon the simple yet effective idea of sending and receiving messages.6 They utilize WebSockets as the default transport mechanism, providing direct two-way communication between client and server, with a fallback to LongPolling if WebSockets are unavailable.6 This persistent connection allows the server to push updates to clients immediately, rather than clients having to poll for changes, drastically reducing the perceived latency for player actions and game state updates. This built-in, efficient real-time communication mechanism directly addresses the "scalability" and "concurrency" requirements of the user's project. It means developers spend less time implementing complex real-time infrastructure and more time focusing on core game logic, as the framework provides industry-leading performance for concurrent, real-time systems.
Key concepts underpinning Phoenix Channels include:
Socket Handlers: These components manage the underlying connection to the server, multiplexing multiple channel sockets over a single persistent connection.6
Topics: Channels are organized around "topics," which are string identifiers (e.g., "room:lobby", "game:123", "chat:general"). Clients subscribe to specific topics to receive messages. Wildcards (*) can be employed for flexible topic matching, allowing a single channel to handle multiple related subtopics.6
Events: Within a topic, messages are identified by "events," which are named strings (e.g., "new_msg", "card_played", "user_joined").6 These events categorize the type of message being sent or received.
PubSub: Phoenix's PubSub (Publish-Subscribe) layer is the underlying mechanism that handles the mechanics of subscribing to topics, unsubscribing, and broadcasting messages across all interested subscribers.6 Channels utilize PubSub internally to perform much of their work, efficiently fanning out messages. The PubSub system is not limited to broadcasting chat messages; it is a powerful and flexible pattern for distributing
any game state changes across all connected clients and even other internal Elixir processes. When a player makes a move, the game's managing process (e.g., a GenServer, covered in Phase 4) updates its internal state and then publishes an event (e.g., "card_played", "turn_ended") to a game-specific PubSub topic. All LiveViews, other Channels (e.g., for spectators), or even AI player processes subscribed to that topic will instantly receive this update. This pattern elegantly decouples the core game logic from the UI rendering and specific client-side concerns, enhancing modularity, allowing for efficient fan-out of updates, and simplifying the architecture for distributing game state in a highly concurrent and potentially distributed environment, making the system more robust and easier to scale.
On the server-side, Phoenix Channels are implemented as Elixir modules that define callbacks to handle various stages of communication. Key callbacks include join/3 (for authorizing clients to a given topic), handle_in/3 (for processing incoming events from clients), and broadcast/3 or push/3 (for sending messages to clients or broadcasting to all subscribers).21
Phoenix ships with its own JavaScript client library for client-side interaction. Developers initialize a Socket connection, then create and join a Channel instance for a specific topic. The channel.push() method is used to send messages to the server, and channel.on() is used to register callbacks for receiving messages from the server.6

Building the Chat Server

A chat server serves as an exemplary use case for demonstrating the power of Phoenix Channels in a real-time multiplayer environment.23 Its implementation showcases the fundamental principles of real-time communication.
Channel Setup: The process begins by generating a new channel module using the mix phx.gen.channel Chat command.21 This command creates the necessary boilerplate code, providing a starting point for the channel's logic.
Server-Side (ChatChannel):
The join/3 callback within the ChatChannel is implemented to authorize users attempting to join a chat room topic (e.g., "room:general" for a lobby chat, or "game:<game_id>" for in-game chat).21 This is the point where authentication logic is integrated, verifying user tokens to ensure that only authorized players can access specific chat rooms.27
The handle_in/3 callback is implemented to process incoming chat messages from clients (e.g., an event named "new_msg"). After processing the message, this function typically uses broadcast! to send the message to all clients subscribed to that chat room's topic, ensuring real-time delivery to all participants.21
For persistent chat history, integrating Ecto to save messages to a database is a common practice.25 This ensures that messages are retained even if clients disconnect or the server restarts.
Client-Side (JavaScript/TypeScript):
The app.js file must initialize the Phoenix Socket and LiveSocket to establish the connection to the server.16
An instance of a channel for the desired chat topic is created using let channel = socket.channel("chat:general", {}).6
The client then joins the channel: channel.join().receive("ok", resp => console.log("Joined successfully", resp)).6
To send messages, the channel.push("new_msg", {body: "Your message here"}) method is utilized.21
To receive messages, an event listener is registered using channel.on("new_msg", payload => { /* Logic to display the incoming message */ }).21 This callback is triggered whenever a message is broadcast to the subscribed topic.

Integrating LiveView for Real-time Game Updates

Phoenix LiveView is perfectly suited for rendering the main game user interface and providing real-time updates as the game state evolves.4 Its server-centric approach simplifies UI development significantly.
The mount/2 callback in the game's LiveView will initialize the LiveView's state, including the initial game state, when a client connects.5 The
render/1 function will then dynamically display the HTML representation of the game based on this current state, ensuring the UI is always up-to-date.5
handle_event/3 callbacks will process user interactions, such as a player clicking on a card to play it or interacting with game controls, translating client-side actions into server-side state changes.5
LiveView can seamlessly subscribe to Phoenix PubSub topics.16 When the core game logic (which will likely reside in a GenServer, covered in Phase 4) updates its state (e.g., after a card is played), it can broadcast an event via PubSub to a game-specific topic. The LiveView for that game room will then receive this PubSub message via its
handle_info callback and update its internal assigns, triggering an efficient re-render of the UI for all connected players.5 This creates a highly responsive and synchronized game experience.

Managing Player Connections and Game Rooms

Effective management of player connections and game rooms is crucial for a multiplayer platform. Each active game instance in the platform can be uniquely identified and correspond to its own Phoenix Channel topic (e.g., "game:<game_id>").23 Players will join the specific channel associated with their game to send actions (e.g., playing a card) and receive real-time updates about the game state.
A separate "Lobby" channel (e.g., "lobby:general") can be used to manage game creation, list available games, and allow players to join existing matches. This provides a central hub for users before they enter a specific game.
Dynamic Supervisors, a key OTP component detailed in Phase 4, are the ideal mechanism to start a new GenServer process (representing a unique game instance) for each new game created by users.7 This ensures that each game is isolated, providing inherent fault tolerance and preventing a crash in one game from affecting others.
Table 3: Phoenix Channel Callbacks & Client-Side Interactions
Phoenix Channel Callback (Server-Side)
Purpose/Description
Corresponding JavaScript Client Method
Example Use Case
join/3 21
Authorize client to join a topic. Returns {:ok, socket} or {:error, reply}.
socket.channel("topic", {}).join() 6
Player joining a game room or chat lobby.
handle_in/3 21
Handle incoming events (messages) from the client. Processes payload and updates state.
channel.push("event_name", payload) 21
Player sending a chat message, playing a card, or making a game move.
handle_out/3 21
Intercept outgoing events before they are sent to the client. Used for modification or logging.
(N/A - Server-side interception of broadcasts/pushes)
Adding metadata to a message, logging outgoing events.
broadcast/3 / broadcast!/3 22
Send a message to all subscribers of the socket's topic. broadcast! raises on failure.
channel.on("event_name", callback) 21
Distributing a new chat message to all users in a room, updating game state for all players.
push/3 22
Send an event directly to the connected client without requiring a prior message from them.
channel.on("event_name", callback) 21
Server pushing a private notification or a specific game update to one player.


Phase 4: Robust Game State Management – GenServer & OTP


Deep Dive into GenServer: Managing Game State for Individual Games

A GenServer (Generic Server) is a foundational building block within the Elixir ecosystem for managing state and encapsulating functionality within a single, isolated process.30 It behaves like a long-running process that can maintain state, execute code asynchronously, and handle various messages, making it ideal for managing the dynamic aspects of a game.31
For a multiplayer card game platform, the most idiomatic and robust approach is to manage each active game (e.g., a specific Hearts match between players) with its own dedicated GenServer process.20 This design ensures strong isolation: a crash or error within one game's
GenServer will not affect other active games or the overall application. This capability is a direct benefit of the BEAM's process model, where failures are contained. This approach allows a GenServer to encapsulate the entire game logic and state for a single match, leveraging process isolation for fault tolerance. This means that each game instance is an independent, self-contained unit, simplifying the management of concurrent games and enhancing the overall resilience of the platform.
GenServer provides a set of callbacks that define its behavior and how it reacts to messages:
init/1: This callback is invoked when the GenServer process starts. It is responsible for setting up the initial state of the process, such as creating a shuffled deck and dealing initial hands to players.31 This function determines the starting conditions of each game instance.
handle_call/3: This callback is used for synchronous requests from clients. When a client makes a GenServer.call (e.g., "What's the current game state?", "Is it my turn?"), this callback processes the request and sends an immediate reply back to the caller, blocking until a response is received.31 This is suitable for queries where an immediate response is expected.
handle_cast/2: This callback is used for asynchronous requests. When a client sends a GenServer.cast (e.g., "Player X played card Y"), this callback processes the message without sending a reply, allowing the caller to continue immediately.31 This is ideal for actions that do not require an immediate acknowledgment.
handle_info/2: This callback handles internal messages or messages from other processes that do not match handle_call or handle_cast patterns. Examples include PubSub messages indicating a player joined the game, a timer ticking for turn timeouts, or messages from other game components.5 This provides a flexible mechanism for internal process communication and event handling.
terminate/2: This optional callback is invoked when the GenServer process is about to exit.22 It allows for cleanup operations or attempting to save the current state to a persistent store. However, it is important to note that
terminate/2 is not guaranteed to be called in cases of abrupt process termination (e.g., a brutal kill or an unhandled crash of the BEAM VM itself).35 Developers must account for this by implementing robust persistence strategies outside of this callback for critical state.
The GenServer's internal state will hold the comprehensive Game struct defined in Phase 2. All game logic functions (playing cards, scoring, turn transitions) will be invoked from within these callbacks. These functions will receive the current Game state, compute the new state based on the action, and return the updated state to the GenServer, which then internally updates its state.3 This functional approach to state updates within a GenServer reinforces immutability and predictability.

OTP Supervision Trees: Ensuring Fault Tolerance for Game Processes

The Open Telecom Platform (OTP) is a powerful set of battle-tested tools and conventions for building concurrent, distributed, and fault-tolerant applications in Elixir.7 Supervision Trees are a core concept within OTP, designed to manage the lifecycle of processes and ensure application stability by handling failures gracefully.7
Supervisors: Supervisors are specialized processes whose sole responsibility is to monitor other processes, known as "child processes".7 When a child process crashes, the supervisor intervenes and restarts it according to a predefined strategy. This mechanism is central to the "let it crash" philosophy in Erlang/Elixir.7 This fundamental philosophy means that instead of trying to prevent every conceivable failure, the system is designed to expect and gracefully recover from crashes. When a process crashes, the supervisor automatically restarts it to a known good state, ensuring the system remains operational. This is immensely beneficial for a multiplayer game, where individual game instances or player connections might encounter transient errors.
Supervision Strategies: OTP provides several restart strategies that dictate how a supervisor reacts to a child process termination 7:
:one_for_one: If a child process crashes, only that specific child is restarted. This is the most common strategy and highly suitable for isolated game instances, ensuring that a problem in one game does not affect others.29
:one_for_all: If any child process crashes, all other child processes are terminated and then all children are restarted. This is useful when processes are tightly coupled and a failure in one implies a corrupted state for all.29
:rest_for_one: If a child process crashes, that process and any processes started after it are restarted. Processes started before the crashed one remain unaffected.29
For a multiplayer card game, the :one_for_one strategy is typically preferred for individual game GenServer processes, as it maximizes isolation and minimizes disruption across the platform.
Dynamic Supervisors: While traditional supervisors start with a predefined list of children, DynamicSupervisor allows for children to be added and removed at runtime.7 This is particularly useful for a multiplayer game platform where new game instances are created dynamically as users start new matches. A
DynamicSupervisor can oversee the creation and management of individual game GenServer processes, ensuring that each new game is properly supervised and resilient to failures. This dynamic management contributes significantly to the system's scalability and fault tolerance by ensuring that each game is isolated and automatically recovered if it crashes.7
Process Registration: To facilitate communication and management, GenServer processes can be registered with names. This allows other processes to interact with them without needing their Process ID (PID). Names can be local to a node (atom), global across a cluster ({:global, term}), or registered via an alternative registry ({:via, module, name}).31 For game instances, registering them by a unique game ID (e.g.,
{:global, "game_#{game_id}"}) allows any connected client or server process to find and interact with that specific game's GenServer.

State Persistence and Recovery

While GenServer processes are excellent for managing in-memory state, this state is lost if the process terminates, especially in cases of abrupt crashes where terminate/2 may not be called.33 Supervisors, by default, restart a child process with its
initial state, not its state prior to the crash.33 Therefore, for critical game state that must survive crashes or application restarts, robust persistence mechanisms are essential.
Several strategies can be employed for state persistence and recovery:
External Storage (Databases): For long-term persistence and complex queries, a traditional database like PostgreSQL is a common choice, integrated via Ecto, Elixir's database wrapper.39 Game state, often represented as a nested map or struct, can be stored in a
JSONB column (for PostgreSQL) or as a binary blob.40 This approach offers strong durability and allows for complex data relationships. When a
GenServer restarts, it can fetch its last known good state from the database during its init/1 callback.38 This strategy offers high durability but introduces network latency for every state read/write.
In-Memory/Disk Caching (ETS and DETS):
ETS (Erlang Term Storage): This is an efficient in-memory key-value store, optimized for Erlang data types. ETS tables can be shared among processes for fast Inter-Process Communication (IPC).42 While extremely fast for reads and writes, ETS tables are non-persistent by default; their data is lost when the owning process terminates or the BEAM VM shuts down.37 A
GenServer can own an ETS table, and its init/1 callback can create or load data into it.37
DETS (Disk Erlang Term Storage): This is a disk-based version of ETS, providing persistence by storing terms on disk using a linear hash table.42 DETS tables are persistent but have a 2GB size limit.42 They can be used for data that rarely changes but is read often, loaded into ETS upon application startup for faster access.45
ETS and DETS are suitable for caching frequently accessed data or for temporary state that does not require full database durability.
Backup Process: A robust pattern for state recovery involves using a separate GenServer whose sole responsibility is to maintain a replica of the primary GenServer's state.38 The primary
GenServer would notify this "backup" process of state changes. If the primary GenServer crashes, its supervisor restarts it, and during its init/1 callback, it can fetch the last known good state from the backup process.38 This approach adds a layer of resilience by decoupling state persistence from the operational logic of the main process.
Libraries for State Persistence: Libraries like Peeper offer an almost drop-in replacement for GenServer that automatically preserves state between crashes.46 Internally,
Peeper creates a specialized supervision tree and manages the persistence, simplifying the developer's task.
The choice of state persistence mechanism depends on the specific requirements for performance, durability, and recovery time. For a multiplayer card game, frequently updated game state might reside primarily in a GenServer's memory, with critical checkpoints or final game results persisted to a database. For high-frequency, low-durability state, ETS could be used, while DETS might serve for less frequently updated, persistent lookup data. This strategic choice of state persistence involves a trade-off between immediate performance and the guarantee of data survival across crashes or restarts. Understanding these options allows for an optimized architecture that balances the need for real-time responsiveness with data integrity.

Phase 5: Advanced Topics & Iterative Refinement


Error Handling & Debugging

Elixir, leveraging the BEAM, provides a distinct and powerful approach to error handling and debugging, deeply integrated with the "let it crash" philosophy. Instead of extensive try/catch blocks for every potential error, the focus shifts to designing processes that are isolated and supervised. When an error occurs within a process, it is allowed to crash, and its supervisor will automatically restart it to a known good state.7 This simplifies the code by removing complex error recovery logic from individual functions.
For monitoring and debugging, Elixir offers several robust tools:
Logger: This module provides a flexible logging framework for recording application events and errors. Well-placed log messages are crucial for understanding system behavior and diagnosing issues in a concurrent environment.
IEx (Interactive Elixir): The interactive shell is invaluable for live debugging. Developers can connect to a running application, inspect process states, call functions, and experiment with code in real-time.10 This is particularly useful for understanding the dynamic state of
GenServer processes.
observer: This graphical tool, part of Erlang/OTP, provides a visual representation of the running BEAM system. It allows developers to inspect processes, applications, and supervision trees, monitor resource usage, and trace messages between processes. This offers a high-level view of the system's health and helps pinpoint bottlenecks or misbehaving processes.
The debugging process in Elixir often involves observing crashes, understanding the reason for termination, and then refining the process logic or supervision strategy to ensure correct recovery or prevention of the specific error. The immutability of data structures also aids debugging, as the state at any given point is a snapshot, making it easier to trace the flow of data and identify where an issue might have originated.

Deployment Considerations

Deploying Elixir and Phoenix applications leverages the BEAM VM's capabilities for robust, long-running systems.
Releases: For production deployments, Elixir applications are typically packaged as "releases." A release is a self-contained executable that includes the BEAM VM, the application code, and all its dependencies, making deployment straightforward without requiring a full Elixir installation on the target server.47
Hot Code Swapping: One of BEAM's most compelling features is "hot code swapping," which allows developers to update running applications' code without causing downtime.1 This is achieved by loading new versions of modules into the running system. While powerful, it requires careful design and adherence to specific upgrade procedures to ensure smooth transitions and state migration, especially for stateful processes like
GenServer.32 It is generally not recommended to perform hot code swaps during development (e.g., via
recompile()) as it can lead to inconsistent states.47
Scaling Strategies: Elixir and Phoenix applications are inherently designed for scalability.
Vertical Scaling: The BEAM VM effectively utilizes multiple CPU cores on a single machine, allowing a single application instance to handle a large number of concurrent processes.2
Horizontal Scaling: Elixir applications can be easily distributed across multiple nodes (machines) in a cluster. BEAM's built-in distribution mechanisms allow processes on different nodes to communicate as if they were on the same machine, facilitating seamless scaling.30 This is particularly beneficial for a multiplayer game platform, where game instances can be distributed across a cluster to handle a growing number of concurrent matches. Load balancers can direct players to the closest or least-loaded server, minimizing latency.48

Testing Strategy

Elixir's functional programming paradigm and immutable data structures greatly simplify the testing process, making it an integral and efficient part of the development workflow.3
Unit Tests for Pure Functions: Pure functions, which produce the same output for the same input and have no side effects, are inherently easy to test in isolation. ExUnit, Elixir's built-in testing framework, is used to write concise and effective unit tests for core game logic functions (e.g., shuffling, dealing, scoring calculations).3 This ensures the accuracy and reliability of individual game mechanics.
Integration Tests for GenServers and Channels: Testing stateful processes like GenServer and real-time components like Phoenix Channels requires a different approach. Integration tests can simulate client interactions, send messages to GenServer processes, and assert on the resulting state changes or messages broadcast through channels. Elixir's concurrency model makes it straightforward to set up isolated test environments where multiple processes can interact without interference.
LiveView Tests for UI Interactions: Phoenix LiveView provides its own testing utilities that allow developers to simulate user interactions (e.g., clicks, form submissions) and assert on the rendered HTML and the LiveView's state changes. This enables comprehensive testing of the interactive user interface without needing a full browser environment, accelerating the feedback loop for UI development.
The inherent immutability in Elixir simplifies test setup and tear-down, as test cases do not need to worry about shared mutable state or side effects from previous tests. This contributes to faster, more reliable, and easier-to-maintain test suites.

AI/LLM for Refinement

AI/LLMs can continue to serve as powerful accelerators throughout the refinement stages of the project. Their utility extends beyond initial learning to assist in advanced development tasks:
Code Generation: Generating boilerplate for new GenServer callbacks, Phoenix Channels, or LiveView components.
Refactoring Suggestions: Providing alternative, more idiomatic Elixir patterns for existing code, or suggesting ways to improve function purity and modularity.
Performance Optimization Tips: Analyzing code snippets and suggesting potential areas for performance improvements, especially concerning concurrent operations or data structure choices.
Exploring Advanced Patterns: Assisting in understanding and implementing more complex OTP behaviors, distributed system patterns, or advanced LiveView features.
While AI can significantly boost productivity and learning, critical evaluation of its output remains paramount. The developer's understanding of Elixir's core principles, the BEAM, and OTP is essential to effectively leverage AI suggestions and ensure the generated code aligns with the project's architectural goals for scalability, concurrency, and fault tolerance.

Conclusion

The journey of building a web-based multiplayer card game platform with Elixir and Phoenix offers a profound learning experience for JavaScript/TypeScript developers. The analysis demonstrates that Elixir and Phoenix are not merely alternative web technologies but represent a powerful paradigm shift, particularly advantageous for real-time, concurrent, and fault-tolerant applications.
The BEAM VM's inherent capabilities for lightweight processes, asynchronous message passing, and the "let it crash" philosophy, combined with OTP's supervision trees, provide a robust foundation for systems that can gracefully handle failures and scale effectively. Phoenix Channels and LiveView further extend these strengths to the web layer, simplifying the development of interactive user interfaces and real-time communication that would be significantly more complex in traditional environments. The functional programming paradigm, with its emphasis on immutability and pattern matching, naturally leads to more predictable, maintainable, and testable game logic.
For continued learning and project expansion, several recommendations are pertinent:
Embrace Iteration: Continue to build features incrementally, applying new Elixir and Phoenix concepts as needed. Start with a single card game (e.g., Hearts) and gradually add others, using the experience gained to refine the game logic and state management.
Deepen OTP Understanding: While this plan introduces GenServer and Supervisors, a deeper dive into OTP behaviors, including DynamicSupervisor for managing a multitude of game instances, will be invaluable for scaling the platform. Explore how different supervision strategies can be applied to various components of the game.
Refine State Persistence: As the game evolves, re-evaluate the state persistence strategy. For critical game state, consider a hybrid approach: maintain active game state in GenServer memory for performance, but periodically persist key checkpoints or final game results to a robust external database like PostgreSQL. For rapidly changing, non-critical data, ETS can provide high-speed in-memory caching.
Leverage the Community and Documentation: The Elixir community is highly supportive, and the official documentation for Elixir and Phoenix is extensive. These resources, alongside AI/LLM assistance, will be crucial for overcoming challenges and discovering idiomatic solutions.
Focus on Testing: Maintain a strong testing discipline. The functional nature of Elixir makes testing straightforward, and comprehensive test suites will ensure the reliability and correctness of complex game logic and real-time interactions.
By following this iterative plan, a JS/TS developer can not only build a sophisticated multiplayer card game platform but also acquire a deep, practical understanding of Elixir and Phoenix, positioning them to tackle a wide range of concurrent and fault-tolerant system challenges.
Works cited
The Complete Guide to Elixir and the BEAM VM in Fault-Tolerant Systems | AppMaster, accessed June 22, 2025, https://appmaster.io/blog/elixir-and-the-beam-vm-in-fault-tolerant-systems
The BEAM-Erlang's virtual machine -, accessed June 22, 2025, https://www.erlang-solutions.com/blog/the-beam-erlangs-virtual-machine/
Brewing Board Games With Elixir - DockYard, accessed June 22, 2025, https://dockyard.com/blog/2024/10/10/brewing-board-games-with-elixir
Understanding the Structure of a Phoenix LiveView Application - DEV Community, accessed June 22, 2025, https://dev.to/msnmongare/understanding-the-structure-of-a-phoenix-liveview-application-1l8p
Phoenix LiveView tutorial | Curiosum, accessed June 22, 2025, https://curiosum.com/blog/phoenix-live-view-tutorial
phoenix_guides-examples/channels.md at master - GitHub, accessed June 22, 2025, https://github.com/jeffkreeftmeijer/phoenix_guides-examples/blob/master/channels.md
Exploring Elixir's OTP Supervision Trees - CloudDevs, accessed June 22, 2025, https://clouddevs.com/elixir/otp-supervision-trees/
How to build a Concurrent & Resilient Service in Elixir - Blog - Finiam, accessed June 22, 2025, https://blog.finiam.com/blog/how-to-build-concurrent-and-resilient-service-in-elixir
Where should game state stuff live for a multiplayer card game? : r/unrealengine - Reddit, accessed June 22, 2025, https://www.reddit.com/r/unrealengine/comments/17qrya2/where_should_game_state_stuff_live_for_a/
A Beginner's Guide to Installing Elixir with asdf | Gigalixir, accessed June 22, 2025, https://www.gigalixir.com/blog/a-beginners-guide-to-installing-elixir-with-asdf/
Install - The Elixir programming language, accessed June 22, 2025, https://elixir-lang.org/install.html
Phoenix for Rails Developers: A Practical Example - Part 1 - Rootstrap, accessed June 22, 2025, https://www.rootstrap.com/blog/phoenix-for-rails-developers-a-practical-example-part-1
Program a simple game with Elixir | Opensource.com, accessed June 22, 2025, https://opensource.com/article/20/12/elixir
Map — Elixir v1.12.3 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/elixir/1.12/Map.html
Structs — Elixir v1.18.4 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/elixir/structs.html
LiveView — Phoenix v1.8.0-rc.3 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/phoenix/1.8.0-rc.3/live_view.html
Poker hand analyser - Rosetta Code, accessed June 22, 2025, https://rosettacode.org/wiki/Poker_hand_analyser
CardDeck — card_deck v0.1.0 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/card_deck/CardDeck.html
Guidance to implement war card game (Elixir) - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/75568946/guidance-to-implement-war-card-game-elixir
Managing 2 states in 1 GenServer - Questions / Help - Elixir Programming Language Forum, accessed June 22, 2025, https://elixirforum.com/t/managing-2-states-in-1-genserver/12185
How To: Use Phoenix Channels | CodeCast, accessed June 22, 2025, https://info.codecast.io/blog/how-to-use-phoenix-channels
Phoenix.Channel — Phoenix v1.7.21 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/phoenix/Phoenix.Channel.html
How to Use Phoenix Websockets with Swift on iOS - ⚡️ Blixt Dev ⚡️, accessed June 22, 2025, https://blixtdev.com/how-to-use-phoenix-websockets-with-swift-on-ios/
phoenix 1.7.21 | Documentation - HexDocs, accessed June 22, 2025, https://hexdocs.pm/phoenix/js/
dwyl/phoenix-chat-example: The Step-by-Step Beginners Tutorial for Building, Testing & Deploying a Chat app in Phoenix 1.7 [Latest] - GitHub, accessed June 22, 2025, https://github.com/dwyl/phoenix-chat-example
Building a Chat Application with Elixir and Phoenix Channels - CloudDevs, accessed June 22, 2025, https://clouddevs.com/elixir/chat-application-with-phoenix-channels/
Phoenix.Token — Phoenix v1.7.21 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/phoenix/Phoenix.Token.html
Implement PubSub with Phoenix LiveView Like a Pro: Comprehensive Guide - YouTube, accessed June 22, 2025, https://www.youtube.com/watch?v=i49AaypvVt0
OTP Supervisors - Elixir School, accessed June 22, 2025, https://elixirschool.com/en/lessons/advanced/otp_supervisors
Managing Distributed State with GenServers in Phoenix and Elixir | AppSignal Blog, accessed June 22, 2025, https://blog.appsignal.com/2024/10/29/managing-distributed-state-with-genservers-in-phoenix-and-elixir.html
GenServer - RunElixir.com, accessed June 22, 2025, https://runelixir.com/genserver.html
Mastering GenServer for Enhanced Elixir Applications - Curiosum, accessed June 22, 2025, https://curiosum.com/blog/what-is-elixir-genserver
GenServer — Elixir v1.15.3 - HexDocs, accessed June 22, 2025, https://hexdocs.pm/elixir/1.15.3/GenServer.html
GenServer behaviour (Elixir v1.18.4) - HexDocs, accessed June 22, 2025, https://hexdocs.pm/elixir/GenServer.html
Graceful shutdown of GenServer - elixir - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/39756769/graceful-shutdown-of-genserver
What are the strategies available to supervisors in Elixir? - Educative.io, accessed June 22, 2025, https://www.educative.io/answers/what-are-the-strategies-available-to-supervisors-in-elixir
Initialize ETS cache using GenServer - elixir - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/40534821/initialize-ets-cache-using-genserver
Saving and restoring process state after crash - Questions / Help - Elixir Forum, accessed June 22, 2025, https://elixirforum.com/t/saving-and-restoring-process-state-after-crash/37537
Sharing state between genservers : r/elixir - Reddit, accessed June 22, 2025, https://www.reddit.com/r/elixir/comments/zzv5mm/sharing_state_between_genservers/
Best way to persist a processes state? - Elixir Forum, accessed June 22, 2025, https://elixirforum.com/t/best-way-to-persist-a-processes-state/68404
Elixir: restart GenServer with certain action - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/42924580/elixir-restart-genserver-with-certain-action
ETS, Mnesia, Riak, oh my! : r/elixir - Reddit, accessed June 22, 2025, https://www.reddit.com/r/elixir/comments/3zljrt/ets_mnesia_riak_oh_my/
which is more efficent among ets and mnesia - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/28493453/which-is-more-efficent-among-ets-and-mnesia
HashDict and OTP GenServer context within Elixir - Stack Overflow, accessed June 22, 2025, https://stackoverflow.com/questions/34271159/hashdict-and-otp-genserver-context-within-elixir
Hydrate ETS from DETS using an init GenServer - Livebook Notebook - free sample from Elixir Patterns book - GitHub Gist, accessed June 22, 2025, https://gist.github.com/hugobarauna/f6268bfe93ae8a19d0fba03f20d1b897
README.md - Hex Preview, accessed June 22, 2025, https://preview.hex.pm/preview/peeper/show/README.md
weird state with gen_server changes on erlang 20.2 and elixir 1.6.1 · Issue #7399 - GitHub, accessed June 22, 2025, https://github.com/elixir-lang/elixir/issues/7399
Guidance needed: is Elixir a good fit for this project? - Reddit, accessed June 22, 2025, https://www.reddit.com/r/elixir/comments/1i9kdlm/guidance_needed_is_elixir_a_good_fit_for_this/
