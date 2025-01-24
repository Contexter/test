```

openapi: 3.1.0
info:
  title: Central Sequence Service API
  description: >
    This API manages the assignment and updating of sequence numbers for various elements within a story, ensuring logical order and consistency. The service persists data to an SQLite database, which is then synchronized with Typesense for real-time search and retrieval capabilities.
  version: 4.0.0
servers:
  - url: https://centralsequence.fountain.coach
    description: Production server for Central Sequence Service API
  - url: https://staging.centralsequence.fountain.coach
    description: Staging server
paths:
  /sequence:
    post:
      summary: Generate Sequence Number
      operationId: generateSequenceNumber
      tags:
        - Sequence Management
      description: Generates a new sequence number for a specified element type, persists it to an SQLite database, and synchronizes it with Typesense. If synchronization with Typesense fails, a retry mechanism will be triggered automatically.
      requestBody:
        required: true
        description: Details of the element requesting a sequence number.
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SequenceRequest'
      responses:
        '201':
          description: Sequence number successfully generated and synchronized.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SequenceResponse'
        '400':
          description: Invalid request parameters.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '502':
          description: Failed to synchronize with Typesense.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TypesenseErrorResponse'
        '500':
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
  /sequence/reorder:
    put:
      summary: Reorder Elements
      operationId: reorderElements
      tags:
        - Sequence Management
      description: Reorders elements by updating their sequence numbers, persists the changes to an SQLite database, and synchronizes the changes with Typesense. If synchronization with Typesense fails, a retry mechanism will be triggered automatically.
      requestBody:
        required: true
        description: Details of the reordering request.
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ReorderRequest'
      responses:
        '200':
          description: Elements successfully reordered and synchronized.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ReorderResponse'
        '400':
          description: Invalid request parameters.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '502':
          description: Failed to synchronize with Typesense.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TypesenseErrorResponse'
        '500':
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
  /sequence/version:
    post:
      summary: Create New Version
      operationId: createVersion
      tags:
        - Version Management
      description: Creates a new version of an element, persists it to an SQLite database, and synchronizes it with Typesense. If synchronization with Typesense fails, a retry mechanism will be triggered automatically.
      requestBody:
        required: true
        description: Details of the versioning request.
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/VersionRequest'
      responses:
        '201':
          description: New version successfully created and synchronized.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/VersionResponse'
        '400':
          description: Invalid request parameters.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '502':
          description: Failed to synchronize with Typesense.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TypesenseErrorResponse'
        '500':
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
components:
  schemas:
    SequenceRequest:
      description: Schema representing a request to generate a new sequence number
      type: object
      properties:
        elementType:
          type: string
          description: Type of the element (e.g., script, section, character, action, spokenWord).
          enum: [script, section, character, action, spokenWord]
        elementId:
          type: integer
          description: Unique identifier of the element.
          minimum: 1
        comment:
          type: string
          description: Contextual explanation for generating the sequence number.
      required: [elementType, elementId, comment]
    SequenceResponse:
      description: Schema representing the response with a generated sequence number
      type: object
      properties:
        sequenceNumber:
          type: integer
          description: The generated sequence number.
          minimum: 1
        comment:
          type: string
          description: Contextual explanation generated dynamically by the GPT model, explaining why the sequence number was generated.
    ReorderRequest:
      description: Schema representing a request to reorder elements
      type: object
      properties:
        elementType:
          type: string
          description: Type of elements being reordered.
          enum: [script, section, character, action, spokenWord]
        elements:
          type: array
          items:
            type: object
            properties:
              elementId:
                type: integer
                description: Unique identifier of the element.
                minimum: 1
              newSequence:
                type: integer
                description: New sequence number for the element.
                minimum: 1
        comment:
          type: string
          description: Contextual explanation for reordering the elements.
      required: [elementType, elements, comment]
    ReorderResponse:
      description: Schema representing the response after reordering elements
      type: object
      properties:
        updatedElements:
          type: array
          items:
            type: object
            properties:
              elementId:
                type: integer
                description: Unique identifier of the element.
              newSequence:
                type: integer
                description: Updated sequence number.
        comment:
          type: string
          description: Contextual explanation generated dynamically by the GPT model, explaining why the elements were reordered.
    VersionRequest:
      description: Schema representing a request to create a new version of an element
      type: object
      properties:
        elementType:
          type: string
          description: Type of the element (e.g., script, section, character, action, spokenWord).
          enum: [script, section, character, action, spokenWord]
        elementId:
          type: integer
          description: Unique identifier of the element.
          minimum: 1
        newVersionData:
          type: object
          description: Data for the new version of the element.
        comment:
          type: string
          description: Contextual explanation for creating the new version.
      required: [elementType, elementId, newVersionData, comment]
    VersionResponse:
      description: Schema representing the response with the new version number
      type: object
      properties:
        versionNumber:
          type: integer
          description: The version number of the new version.
          minimum: 1
        comment:
          type: string
          description: Contextual explanation generated dynamically by the GPT model, explaining why the new version was created.
    SuccessResponse:
      type: object
      properties:
        message:
          type: string
          description: Success message.
    ErrorResponse:
      type: object
      properties:
        errorCode:
          type: string
          description: Application-specific error code.
        message:
          type: string
          description: Human-readable error message.
        details:
          type: string
          description: Additional information about the error, if available.
    TypesenseErrorResponse:
      type: object
      properties:
        errorCode:
          type: string
          description: Error code related to Typesense synchronization.
        retryAttempt:
          type: integer
          description: Number of retry attempts made to synchronize with Typesense.
        message:
          type: string
          description: Human-readable error message.
        details:
          type: string
          description: Additional information about the Typesense error, if available.
  securitySchemes:
    apiKeyAuth:
      type: apiKey
      in: header
      name: X-API-KEY
security:
  - apiKeyAuth: []
```

