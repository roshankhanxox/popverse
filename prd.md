# **Product Requirements Document: PopVerse Rap Battle Farcaster Mini-App**

Document Version: 1.0  
Date: July 22, 2025  
Product Manager: \[Your Name/Team\]  
Development Team: Windsurf

## **1\. Introduction & Overview**

The "PopVerse Rap Battle" is an interactive Farcaster mini-app inspired by "MadVerse City UK: The Rap Game." It aims to bring the excitement of rap battles directly into the Farcaster feed, allowing users to engage in turn-based lyrical showdowns. Users will submit rhymes, which will be converted to speech using ElevenLabs, overlaid on a beat, and then presented for community voting.  
This document outlines the core features, user flows, and technical considerations required to develop the initial version of the PopVerse mini-app.

## **2\. Goals**

* **Engage Farcaster Users:** Create a highly interactive and entertaining experience that encourages participation and virality within the Farcaster ecosystem.  
* **Showcase Onchain Interactivity:** Demonstrate the power of Farcaster Frames for dynamic, multi-user applications.  
* **Leverage Modern APIs:** Effectively integrate ElevenLabs for TTS and Supabase for real-time data and storage to deliver a rich user experience.  
* **Establish a Foundational Platform:** Build a robust core application that can be expanded with future features (e.g., AI judge, user profiles, tournaments).

## **3\. Target Audience**

* **Farcaster Users:** Individuals active on Farcaster who enjoy interactive content, social gaming, and creative expression.  
* **Rap/Music Enthusiasts:** Users with an interest in rap music, lyrical battles, and creative writing.  
* **Early Adopters of Onchain Apps:** Users keen to explore innovative applications on decentralized social platforms.

## **4\. Key Features**

### **4.1. Battle Initiation**

* **Start Battle Button:** A clear button on the initial frame allowing any user to initiate a new rap battle.  
* **Player Assignment:** The user who initiates the battle becomes "Player 1." The app will then prompt for a "Player 2." This can be done by allowing another user to "join" the battle, or by the initiator challenging a specific FID. For V1, we will assume a "join" mechanism where any user can click "Join Battle" on the next frame to become Player 2\.

### **4.2. Rhyme Submission**

* **Text Input Field:** A dedicated text input field within the Farcaster Frame for players to type their rhymes.  
* **Character Limit:** A reasonable character limit (e.g., 280 characters, similar to a tweet) for rhyme submissions to fit within frame constraints and encourage conciseness.  
* **Submit Button:** A button to submit the entered rhyme.  
* **Turn-Based System:** The app will enforce a strict turn-based system, ensuring only the current player can submit their rhyme.

### **4.3. Audio Generation & Playback**

* **ElevenLabs Integration:** Automatically convert submitted rhyme text into high-quality speech audio using the ElevenLabs API upon submission.  
* **Beat Overlay:** Overlay the generated speech audio onto a pre-selected generic hip-hop beat. This mixing will occur server-side.  
* **Audio Hosting:** Store the mixed audio files on Supabase Storage and provide publicly accessible URLs.  
* **In-Frame Audio Player/Link:** The Farcaster Frame will display a mechanism for users to play the generated audio for each rhyme. This could be an embedded audio player (if supported by the Farcaster client) or a direct link to the audio file.

### **4.4. Voting Mechanism**

* **Listen & Vote Frame:** After both players have submitted their rhymes, a new frame will appear, allowing all Farcaster users to listen to both rhymes.  
* **Vote Buttons:** Clear buttons (e.g., "Vote for Player 1," "Vote for Player 2") for users to cast their vote.  
* **One Vote Per User Per Battle:** Enforce that each Farcaster ID can only cast one vote per battle.  
* **Real-time Vote Count (Optional for V1, good for V2):** While frames are stateless, the server can re-render the frame on each vote to show updated vote counts. For V1, simply acknowledging the vote might suffice.

### **4.5. Battle Conclusion & Winner Announcement**

* **Vote Tally:** After a predefined voting period (e.g., 5 minutes from the last rhyme submission, or after a certain number of votes), the server will tally the votes.  
* **Winner Display:** A final Farcaster Frame announcing the winner of the battle based on the vote count.  
* **Tie-breaking (V2):** Define a tie-breaking rule (e.g., first to reach a certain vote count, or a random selection).

## **5\. User Flows**

### **5.1. Starting a New Battle**

1. User sees the initial "PopVerse Rap Battle" frame.  
2. User clicks "Start Battle" button.  
3. Server receives interaction, validates, creates new battle entry in Supabase.  
4. Server returns a new frame: "Player 1 (You\!), enter your rhyme below:" with text input and "Submit Rhyme" button.

### **5.2. Player 1 Submits Rhyme**

1. Player 1 types rhyme into the text input.  
2. Player 1 clicks "Submit Rhyme."  
3. Server receives rhyme, calls ElevenLabs for TTS.  
4. Server mixes TTS audio with a generic beat.  
5. Server uploads mixed audio to Supabase Storage, gets audio\_url.  
6. Server updates rhymes table with rhyme text and audio\_url, updates battle status to p1\_rhyme\_submitted.  
7. Server returns a new frame: "Player 1's rhyme submitted\! Waiting for Player 2 to join/submit." (This frame might include a "Listen to P1's Rhyme" button/link).

### **5.3. Player 2 Joins/Submits Rhyme**

1. Another user sees the "Waiting for Player 2" frame and clicks "Join Battle" (or is explicitly challenged).  
2. Server assigns the joining user as Player 2, updates battle entry.  
3. Server returns a new frame for Player 2: "Player 2 (You\!), enter your rhyme below:" with text input and "Submit Rhyme" button.  
4. Player 2 types rhyme and clicks "Submit Rhyme."  
5. Server processes Player 2's rhyme similarly to Player 1 (ElevenLabs, mixing, storage, DB update).  
6. Server updates battle status to p2\_rhyme\_submitted and then voting.  
7. Server returns the "Battle Ready\! Listen & Vote\!" frame with links/players for both rhymes and voting buttons.

### **5.4. Voting**

1. Any Farcaster user sees the "Battle Ready\! Listen & Vote\!" frame.  
2. User listens to Player 1's and Player 2's rhymes.  
3. User clicks "Vote for Player 1" or "Vote for Player 2."  
4. Server receives vote, validates (one vote per user per battle), records in votes table.  
5. Server returns an updated frame (e.g., "Thanks for your vote\!" or showing current vote counts).

### **5.5. Battle Conclusion**

1. After voting period/threshold, server tallies votes from votes table.  
2. Server updates battle status to completed and sets winner\_fid.  
3. Server returns the final frame: "Battle Concluded\! The Winner is \[Winner's Farcaster Username\]\!" with final scores.

## **6\. Technical Architecture (High-Level)**

The core architecture, as detailed in the rap-game-architecture immersive artifact, will be followed.

* **Project Setup:** The project is already initialized using npx create-onchain@latest \--mini.  
* **Farcaster Frame Server (Backend):**  
  * **Technology:** Python (Flask/FastAPI) is preferred for its robust API and audio processing libraries.  
  * **Hosting:** A reliable server environment capable of handling HTTP requests and external API calls.  
  * **Key Libraries:** Farcaster Frames SDK/utilities for signature verification, requests for external APIs, pydub for audio mixing.  
* **Supabase (Backend-as-a-Service):**  
  * **Database:** PostgreSQL for storing battles, rhymes, votes, and users data.  
  * **Storage:** For hosting generated MP3 audio files.  
  * **Realtime (Future):** Potentially for live dashboards.  
* **ElevenLabs API:** For Text-to-Speech generation. API key management will be handled securely on the server.  
* **Generic Beats:** Pre-selected MP3 files hosted on Supabase Storage or the server for mixing.

## **7\. Future Considerations / Phases (V2+)**

* **AI Judge:** Integration of a Large Language Model (e.g., Gemini API) to evaluate rhyme quality and assign scores or determine winners.  
* **User Profiles & Stats:** Displaying user win/loss records, total battles, and top rhymes.  
* **Battle History:** A view for users to browse past battles.  
* **Challenge System:** Allowing users to directly challenge specific Farcaster friends.  
* **Leaderboards:** Ranking users based on wins or judge scores.  
* **Custom Beats:** Option for users to select from a wider library of beats.  
* **Monetization (Optional):** Exploring mechanisms like premium beats or battle entry fees.  
* **Improved Audio Playback:** Exploring more seamless in-frame audio playback solutions or dedicated web experiences.

## **8\. Success Metrics**

* **Number of New Battles Initiated:** Track daily/weekly battle creations.  
* **Number of Completed Battles:** Measure the completion rate of battles.  
* **Unique Participating Users:** Count distinct Farcaster IDs engaging in battles (submitting rhymes, voting).  
* **Engagement Rate:** Ratio of users interacting with frames to those who view them.  
* **Retention:** How many users return to play multiple battles.  
* **API Usage:** Monitor ElevenLabs and Supabase usage to ensure efficiency and manage costs.