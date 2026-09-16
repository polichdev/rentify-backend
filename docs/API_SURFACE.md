# Rentify Backend — API Surface Specification

All endpoints are versioned under the `/api/v1` base URL path.

Interactive OpenAPI documentation is available via Swagger UI when running locally:
- **Swagger UI**: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
- **OpenAPI 3.0 JSON Spec**: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)

---

## Table of Contents
- [Authentication & Sessions (`/api/v1`)](#authentication--sessions)
- [User Profiles (`/api/v1/users`)](#user-profiles)
- [Properties & Search (`/api/v1/properties`)](#properties--search)
- [Calendar & Availability (`/api/v1/properties`)](#calendar--availability)
- [Bookings (`/api/v1/bookings`)](#bookings)
- [Payments & Wallet (`/api/v1/payments`, `/api/v1/wallet`)](#payments--wallet)
- [Conversations & Messaging (`/api/v1/conversations`)](#conversations--messaging)
- [Reviews & Ratings (`/api/v1/reviews`, `/api/v1/properties/{id}/reviews`)](#reviews--ratings)
- [Favorites (`/api/v1/users/me/favorites`)](#favorites)
- [Locations & Amenities (`/api/v1/locations`, `/api/v1/amenities`)](#locations--amenities)
- [Promotions & Subscriptions (`/api/v1/promotion-packages`, `/api/v1/subscription-purchases`)](#promotions--subscriptions)

---

## Authentication & Sessions

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/users` | Register a new user account | No | `RegisterRequestDto` (email, password, firstName, lastName, phone, role) | `AuthenticationResponseDto` (token, user info) |
| `POST` | `/api/v1/sessions` | Login with email and password | No | `AuthenticationRequestDto` (email, password) | `AuthenticationResponseDto` (token, user info) |
| `POST` | `/api/v1/sessions/google` | Social login via Google OAuth 2.0 | No | `GoogleOAuthRequestDto` (idToken) | `AuthenticationResponseDto` (token, user info) |
| `DELETE` | `/api/v1/sessions/current` | Logout current session | Yes | None | `204 No Content` |

> [!NOTE]
> When `AUTH_STRATEGY=cookie` is enabled, authentication tokens are set via `HttpOnly`, `SameSite=Lax` cookies, and a `CSRF-Token` cookie is issued for mutation verification. When `AUTH_STRATEGY=bearer`, standard `Authorization: Bearer <token>` headers are expected.

---

## User Profiles

Base path: `/api/v1/users`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `GET` | `/api/v1/users/me` | Retrieve authenticated user profile | Yes | None | `UserResponseDto` |
| `GET` | `/api/v1/users/{userId}` | Get public user profile (name, avatar, rating, stats) | No | `userId` (path parameter) | `PublicUserProfileDto` |
| `PUT` | `/api/v1/users/me` | Update editable profile fields | Yes | `UpdateUserRequestDto` | `UserResponseDto` |
| `PATCH` | `/api/v1/users/me/password` | Change account password | Yes | `ChangePasswordRequestDto` (currentPassword, newPassword) | `204 No Content` |
| `DELETE` | `/api/v1/users/me` | Deactivate account and invalidate sessions | Yes | Header: `X-Current-Password` (optional) | `204 No Content` |
| `POST` | `/api/v1/users/me/avatar` | Upload and set profile avatar photo | Yes | Multipart `file` | `String` (Avatar CDN URL) |
| `DELETE` | `/api/v1/users/me/avatar` | Remove user avatar | Yes | None | `204 No Content` |

---

## Properties & Search

Base path: `/api/v1/properties`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `GET` | `/api/v1/properties` | Faceted search and public catalog listing | No | Query: `cityId`, `districtId`, `minPrice`, `maxPrice`, `guests`, `bedrooms`, `amenityIds`, `propertyType`, `checkIn`, `checkOut`, `page`, `size`, `sort` | `Page<PropertyCardDto>` |
| `GET` | `/api/v1/properties/my` | List host's own properties | Yes (Host) | Query: `statuses` (e.g. `ACTIVE,DRAFT`), `page`, `size` | `Page<PropertyCardDto>` |
| `GET` | `/api/v1/properties/{id}` | Get full property listing details | No | `id` (path parameter) | `PropertyDetailsDto` |
| `POST` | `/api/v1/properties` | Create a new property listing | Yes (Host) | `PropertyCreateDto` (title, description, price, address, amenities, rules) | `PropertyDetailsDto` |
| `PUT` | `/api/v1/properties/{id}` | Update existing property listing | Yes (Owner) | `PropertyUpdateDto` | `PropertyDetailsDto` |
| `DELETE` | `/api/v1/properties/{id}` | Delete property listing | Yes (Owner) | `id` (path parameter) | `204 No Content` |
| `PATCH` | `/api/v1/properties/{id}` | Update property status (`DRAFT`, `ACTIVE`, `INACTIVE`, `BLOCKED`) | Yes (Owner) | `PropertyStatusUpdateDto` | `PropertyDetailsDto` |
| `GET` | `/api/v1/properties/map-pins` | Get lightweight geo coordinates for map clustering | No | Query: Same filter params as search | `List<PropertyMapPinDto>` |
| `POST` | `/api/v1/properties/{id}/photos` | Upload property photo to Cloudinary CDN | Yes (Owner) | Multipart `file` | `PropertyPhotoDto` |
| `DELETE` | `/api/v1/properties/{id}/photos/{photoId}` | Delete photo from listing & storage | Yes (Owner) | `id`, `photoId` | `204 No Content` |

---

## Calendar & Availability

Base path: `/api/v1/properties`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/properties/{id}/availability-blocks` | Create manual host block on date range | Yes (Owner) | `AvailabilityBlockRequestDto` (startDate, endDate, reason) | `AvailabilityBlockDto` |
| `GET` | `/api/v1/properties/{propertyId}/availability-blocks` | List host manual availability blocks | Yes (Owner) | `propertyId` | `List<AvailabilityBlockDto>` |
| `DELETE` | `/api/v1/properties/{propertyId}/availability-blocks/{blockId}` | Delete manual availability block | Yes (Owner) | `propertyId`, `blockId` | `204 No Content` |
| `GET` | `/api/v1/properties/{propertyId}/unavailable-date-ranges` | Merged calendar dates unavailable for booking (reservations + manual blocks) | No | `propertyId` | `List<DateRangeDto>` |

---

## Bookings

Base path: `/api/v1/bookings`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/bookings` | Create a new reservation request | Yes (Guest) | `BookingRequestDto` (propertyId, checkIn, checkOut, guestsCount, guestNotes) | `BookingDto` |
| `GET` | `/api/v1/bookings` | Get user bookings (as guest or host) | Yes | Query: `role` (`GUEST` or `HOST`), `status`, `page`, `size` | `Page<BookingDto>` |
| `GET` | `/api/v1/bookings/{id}` | Get reservation details by ID | Yes (Participant) | `id` (path parameter) | `BookingDto` |
| `PATCH` | `/api/v1/bookings/{id}` | Update booking status (`CONFIRMED`, `REJECTED`, `CANCELLED`) | Yes (Participant) | `BookingStatusUpdateRequestDto` (status, reason) | `BookingDto` |

---

## Payments & Wallet

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/payments` | Pay for booking reservation using wallet balance | Yes (Guest) | `PaymentCreateRequestDto` (bookingId) | `PaymentResponseDto` |
| `GET` | `/api/v1/payments` | Get user payments audit history | Yes | None | `List<PaymentResponseDto>` |
| `GET` | `/api/v1/bookings/{bookingId}/payments` | Get payment transactions linked to a booking | Yes (Participant) | `bookingId` (path parameter) | `List<PaymentResponseDto>` |
| `GET` | `/api/v1/wallet` | Get current authenticated user wallet balance & currency | Yes | None | `WalletBalanceDto` |
| `POST` | `/api/v1/wallet/transactions` | Top up wallet balance with specified amount | Yes | `WalletTopUpRequestDto` (amount) | `WalletBalanceDto` |
| `GET` | `/api/v1/wallet/transactions` | Paginated wallet transaction history | Yes | Query: `page`, `size`, `sort` | `Page<WalletTransactionDto>` |
| `GET` | `/api/v1/wallet/transaction-amount-options` | List supported predefined top-up options | Yes | None | `List<TopUpOptionDto>` |

---

## Conversations & Messaging

Base path: `/api/v1/conversations`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/conversations` | Start new conversation thread regarding a property | Yes | `CreateConversationRequestDto` (propertyId, recipientId, initialMessage) | `ConversationDto` |
| `GET` | `/api/v1/conversations` | List conversations of the authenticated user | Yes | None | `List<ConversationDto>` |
| `GET` | `/api/v1/conversations/{conversationId}/messages` | Get message history in thread | Yes (Participant) | `conversationId` | `List<MessageDto>` |
| `POST` | `/api/v1/conversations/{conversationId}/messages` | Send message reply in conversation thread | Yes (Participant) | `SendMessageRequestDto` (text) | `MessageDto` |

---

## Reviews & Ratings

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `POST` | `/api/v1/reviews` | Leave review for a completed stay | Yes (Guest) | `ReviewRequestDto` (bookingId, rating, text, cleanliness, accuracy, communication, location, checkIn, value) | `ReviewDto` |
| `GET` | `/api/v1/properties/{propertyId}/reviews` | Paginated review list for property | No | `propertyId`, Query: `page`, `size`, `sort` | `Page<ReviewDto>` |

---

## Favorites

Base path: `/api/v1/users/me/favorites`

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `GET` | `/api/v1/users/me/favorites` | List authenticated user favorite properties | Yes | None | `List<FavoriteResponseDto>` |
| `PUT` | `/api/v1/users/me/favorites/{propertyId}` | Add property to favorites | Yes | `propertyId` (path parameter) | `FavoriteResponseDto` |
| `DELETE` | `/api/v1/users/me/favorites/{propertyId}` | Remove property from favorites | Yes | `propertyId` (path parameter) | `204 No Content` |

---

## Locations & Amenities

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `GET` | `/api/v1/locations/suggest` | Location auto-suggest search | No | Query: `q` (search term), `cityId`, `types` (`CITY`, `DISTRICT`, `METRO_STATION`, `RESIDENTIAL_COMPLEX`), `limit` | `List<LocationSuggestionDto>` |
| `GET` | `/api/v1/amenities` | List all property amenities | No | Query: `category` (optional) | `List<AmenityDto>` |
| `GET` | `/api/v1/amenities/grouped` | List amenities grouped by category | No | None | `List<AmenityCategoryGroupDto>` |

---

## Promotions & Subscriptions

| Method | Endpoint | Description | Auth Required | Request Body / Params | Response |
|---|---|---|:---:|---|---|
| `GET` | `/api/v1/promotion-packages/top` | List available TOP promotion packages | Yes | None | `List<TopPromotionPackageDto>` |
| `GET` | `/api/v1/promotion-packages/subscriptions` | List available host subscription packages | Yes | None | `List<SubscriptionPackageDto>` |
| `POST` | `/api/v1/properties/{propertyId}/top-promotions` | Buy TOP placement for property | Yes (Owner) | `PurchaseTopPromotionRequestDto` (packageType) | `TopPromotionPurchaseResponseDto` |
| `POST` | `/api/v1/subscription-purchases` | Purchase host subscription upgrade | Yes | `PurchaseSubscriptionRequestDto` (packageType) | `SubscriptionPurchaseResponseDto` |
