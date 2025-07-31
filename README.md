# Channels As Content Flow

> This project was created for my bachelor's degree thesis.

<img width="1683" height="954" alt="slide1-image1 drawio" src="https://github.com/user-attachments/assets/72235bb4-4206-4d2e-a780-5234c5922b8f" />

Over time, social media platforms have revolutionized the way we consume streaming content, shifting from a rigid and predefined flow like that of radio and TV to one that is continuous, personalized, and potentially unlimited.

This has also led to certain issues, such as the lack of uniformity across platforms and compulsive consumption dynamics driven by the attention economy.

## Objectives

<img width="2286" height="666" alt="slide2-image1 drawio" src="https://github.com/user-attachments/assets/40c42d1e-efdb-4e3c-83d4-0c482701a7dd" />

The goal is the development of a multiplatform prototype that allows:

- **Retrieving channels** from YouTube, Twitch, and IPTV
- **Organizing channels** into a personalized user list
- **Accessing content** through official or alternative players

This first version will handle only live content

## Design Choices

The project implements several key architectural patterns and principles:

- **Clean Architecture**
- elements of **Domain-Driven Design** 
- **Model-View-ViewModel (MVVM)** architectural pattern
- **Kotlin Multiplatform** and **Compose Multiplatform** for cross-platform development

This approach enables clear separation of:

- **Presentation logic**
- **Application logic**
- **Business logic**

## Architecture Design

<img width="3745" height="1732" alt="architecture-slide" src="https://github.com/user-attachments/assets/507360a6-f337-4855-a6f4-c0e4313553e7" />

The system is divided into concentric layers, with dependencies pointing toward the model (or core), leveraging the **Dependency Inversion Principle**.

The **Bridge pattern** was extensively used to abstract a generic service across different platforms.

In general, it seems that a service with a certain degree of popularity eventually diverges across different platforms; and the cost of converging them into a single platform is inversely proportional to how similar they are in terms of interfaces or technologies.

---

### Model 

<img width="7235" height="2755" alt="tesi7-Pagina-5 drawio" src="https://github.com/user-attachments/assets/2f675134-dd6f-4eb2-a684-33cfcfe42ca5" />


Core entities:

- **Channel**: A class to represent channels
- **ServiceParameter**: A class to represent the parameter needed to retrieve channels (a token for platforms like YouTube and Twitch, and a URL for IPTV channels)

Both entities have a type that specifies which channel provider is being used.

### Domain 

Contains the use case classes (**Screaming Architecture**) that use the entities in the model through interfaces pointing to the Data layer (outport).

#### Channel Management Use Cases

<img width="3756" height="1341" alt="usecasesgestionecanali" src="https://github.com/user-attachments/assets/f877ef94-6688-4ae7-94ee-17a57dee474c" />

- **GetOAuth2UseCase**: Necessary for retrieving the URL to start the OAuth2 authentication flow
- **FetchChannelsUseCase**: Receives a ServiceParameter and handles retrieving the channels from the providers and saving them locally, based on the type of service parameter provided
- **GetProviderChannelsUseCase**: Retrieves the locally saved channels based on the provided ChannelType

#### User List Management Use Cases

<img width="5256" height="1671" alt="usecasesgestioneuserlist" src="https://github.com/user-attachments/assets/28ee0724-988c-46ea-9484-a374d447ce7a" />

- **UpdateUserListUseCase**: Receives a pair consisting of Channel Position (Int) and Channel, and adds it to the user list. Since it's an update, if a channel already exists at that position, it will be overwritten
- **RemoveFromUserListUseCase**: Receives a channel and removes it from the user list if it is present
- **GetUserListUseCase**: Returns a list of pairs: Channel Position (Int) - Channel
- **ClearUserListUseCase**: Removes all elements from the user list

### Data 

Defines the classes that handle data access and manipulation.

#### Channels Importer

<img width="4083" height="918" alt="channelsimporter" src="https://github.com/user-attachments/assets/6e22273c-1376-4a8a-97f9-a657fc6406cb" />

**ChannelsImporter** acts as a mediator between ChannelsProviderFactory and ChannelsRepositoryFactory:

- Receives a ServiceParameter in its `fetchChannels` method
- Based on the ChannelType of the ServiceParameter, retrieves the appropriate ChannelsProvider and ChannelsRepository
- Fetches the channels from the ChannelsProvider and saves them locally using the ChannelsRepository

#### Channels Repository

<img width="3993" height="1827" alt="channelsrepository" src="https://github.com/user-attachments/assets/b235bec7-f08e-4465-8512-380716937c7a" />

Implementation of the ChannelsRepository (**Abstract Factory + Factory Method + Bridge**):

A common implementation is defined in AbstractChannelsRepository, and the concrete classes will override the factory method to specify which table the operations will be performed on.

#### User List Repository

<img width="3078" height="468" alt="userlistrepository" src="https://github.com/user-attachments/assets/1317dfe3-6abc-4c2f-a26c-3e2e2a4030ea" />

A simple Repository implementation for the user list.

### Framework 

Contains all classes related to infrastructural details.

#### Channels Provider

<img width="4317" height="1710" alt="channelsproviderEchannelsfactory" src="https://github.com/user-attachments/assets/8a289105-e354-46df-a94c-87d1cb83440b" />

Implementation of the ChannelsProvider (**Abstract Factory + Factory Method + Bridge**):

The refined abstractions are the actual API classes; **M3UAPI** is a parser for M3U playlists.

**Data Transfer Object (DTO)** classes are used to translate API responses.

#### Data Access 

<img width="2817" height="1548" alt="DAOeDATABASE" src="https://github.com/user-attachments/assets/6af1cd1e-1190-410f-b19d-b3cdbe3d0d96" />

The **Data Access Objects (DAO)** that actually use the classes generated by SQLDelight to manipulate local data.

### Presentation 

> ⚠️ **Note**: The thesis project focuses on architecture, so the ViewModels and especially the Views built with Compose Multiplatform might not be fully adequate.

---

## Backend

Although all the logic is on the frontend, a backend is required to handle:

- **OAuth2 token exchange** with callback
- **API request support** (such as retrieving the ID of YouTube live streams)
- **Embedded players** - official ones for YouTube and Twitch, and alternative ones for M3U

## Usage Example 


<img width="3716" height="1558" alt="app-use1" src="https://github.com/user-attachments/assets/f281e476-ca66-4861-a24e-744ff98aa64e" />

---

<img width="3717" height="1561" alt="app-use2" src="https://github.com/user-attachments/assets/aa2a7293-b1b9-4d08-ba92-b32b95b884cc" />

## To Do

* a live chat feature also for IPTV channels
* support for cast on TV
* support for on-demand channels (YouTube videos or Twitch VODs)
