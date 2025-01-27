openapi: 3.1.0
info:
  title: Mistral AI API
  description: >-
    Our Chat Completion and Embeddings APIs specification. Create your account
    on [La Plateforme](https://console.mistral.ai) to get access and read the
    [docs](https://docs.mistral.ai) to learn how to use it.
  version: 0.0.2
paths:
  /v1/models:
    get:
      summary: List Models
      description: List all models available to the user.
      operationId: list_models_v1_models_get
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModelList'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - models
  /v1/models/{model_id}:
    get:
      summary: Retrieve Model
      description: Retrieve a model information.
      operationId: retrieve_model_v1_models__model_id__get
      parameters:
        - name: model_id
          in: path
          required: true
          schema:
            type: string
            title: Model Id
          example: ft:open-mistral-7b:587a6b29:20240514:7e773925
          description: The ID of the model to retrieve.
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/BaseModelCard'
                  - $ref: '#/components/schemas/FTModelCard'
                discriminator:
                  propertyName: type
                  mapping:
                    base: '#/components/schemas/BaseModelCard'
                    fine-tuned: '#/components/schemas/FTModelCard'
                title: Response Retrieve Model V1 Models  Model Id  Get
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - models
    delete:
      summary: Delete Model
      description: Delete a fine-tuned model.
      operationId: delete_model_v1_models__model_id__delete
      parameters:
        - name: model_id
          in: path
          required: true
          schema:
            type: string
            title: Model Id
          example: ft:open-mistral-7b:587a6b29:20240514:7e773925
          description: The ID of the model to delete.
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DeleteModelOut'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - models
  /v1/files:
    post:
      operationId: files_api_routes_upload_file
      summary: Upload File
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UploadFileOut'
      description: >-
        Upload a file that can be used across various endpoints.


        The size of individual files can be a maximum of 512 MB. The Fine-tuning
        API only supports .jsonl files.


        Please contact us if you need to increase these storage limits.
      requestBody:
        content:
          multipart/form-data:
            schema:
              title: MultiPartBodyParams
              type: object
              properties:
                file:
                  format: binary
                  title: File
                  type: string
                  description: |-
                    The File object (not file name) to be uploaded.
                     To upload a file and specify a custom file name you should format your request as such:
                     ```bash
                     file=@path/to/your/file.jsonl;filename=custom_name.jsonl
                     ```
                     Otherwise, you can just keep the original file name:
                     ```bash
                     file=@path/to/your/file.jsonl
                     ```
                purpose:
                  $ref: '#/components/schemas/FilePurpose'
              required:
                - file
        required: true
      tags:
        - files
    get:
      operationId: files_api_routes_list_files
      summary: List Files
      parameters:
        - in: query
          name: page
          schema:
            default: 0
            title: Page
            type: integer
          required: false
        - in: query
          name: page_size
          schema:
            default: 100
            title: Page Size
            type: integer
          required: false
        - in: query
          name: sample_type
          schema:
            anyOf:
              - items:
                  $ref: '#/components/schemas/SampleType'
                type: array
              - type: 'null'
            title: Sample Type
          required: false
        - in: query
          name: source
          schema:
            anyOf:
              - items:
                  $ref: '#/components/schemas/Source'
                type: array
              - type: 'null'
            title: Source
          required: false
        - in: query
          name: search
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Search
          required: false
        - in: query
          name: purpose
          schema:
            anyOf:
              - $ref: '#/components/schemas/FilePurpose'
              - type: 'null'
          required: false
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ListFilesOut'
      description: Returns a list of files that belong to the user's organization.
      tags:
        - files
  /v1/files/{file_id}:
    get:
      operationId: files_api_routes_retrieve_file
      summary: Retrieve File
      parameters:
        - in: path
          name: file_id
          schema:
            title: File Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RetrieveFileOut'
      description: Returns information about a specific file.
      tags:
        - files
    delete:
      operationId: files_api_routes_delete_file
      summary: Delete File
      parameters:
        - in: path
          name: file_id
          schema:
            title: File Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DeleteFileOut'
      description: Delete a file.
      tags:
        - files
  /v1/files/{file_id}/content:
    get:
      operationId: files_api_routes_download_file
      summary: Download File
      parameters:
        - in: path
          name: file_id
          schema:
            title: File Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/octet-stream:
              schema:
                type: string
                format: binary
      description: Download a file
      tags:
        - files
  /v1/files/{file_id}/url:
    get:
      operationId: files_api_routes_get_signed_url
      summary: Get Signed Url
      parameters:
        - in: path
          name: file_id
          schema:
            title: File Id
            type: string
          required: true
        - in: query
          name: expiry
          schema:
            default: 24
            description: Number of hours before the url becomes invalid. Defaults to 24h
            title: Expiry
            type: integer
          required: false
          description: Number of hours before the url becomes invalid. Defaults to 24h
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FileSignedURL'
      tags:
        - files
  /v1/fine_tuning/jobs:
    get:
      operationId: jobs_api_routes_fine_tuning_get_fine_tuning_jobs
      summary: Get Fine Tuning Jobs
      parameters:
        - in: query
          name: page
          schema:
            default: 0
            title: Page
            type: integer
          required: false
          description: The page number of the results to be returned.
        - in: query
          name: page_size
          schema:
            default: 100
            title: Page Size
            type: integer
          required: false
          description: The number of items to return per page.
        - in: query
          name: model
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Model
          required: false
          description: >-
            The model name used for fine-tuning to filter on. When set, the
            other results are not displayed.
        - in: query
          name: created_after
          schema:
            anyOf:
              - format: date-time
                type: string
              - type: 'null'
            title: Created After
          required: false
          description: >-
            The date/time to filter on. When set, the results for previous
            creation times are not displayed.
        - in: query
          name: created_by_me
          schema:
            default: false
            title: Created By Me
            type: boolean
          required: false
          description: >-
            When set, only return results for jobs created by the API caller.
            Other results are not displayed.
        - in: query
          name: status
          schema:
            anyOf:
              - enum:
                  - QUEUED
                  - STARTED
                  - VALIDATING
                  - VALIDATED
                  - RUNNING
                  - FAILED_VALIDATION
                  - FAILED
                  - SUCCESS
                  - CANCELLED
                  - CANCELLATION_REQUESTED
                type: string
              - type: 'null'
            title: Status
          required: false
          description: >-
            The current job state to filter on. When set, the other results are
            not displayed.
        - in: query
          name: wandb_project
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Wandb Project
          required: false
          description: >-
            The Weights and Biases project to filter on. When set, the other
            results are not displayed.
        - in: query
          name: wandb_name
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Wandb Name
          required: false
          description: >-
            The Weight and Biases run name to filter on. When set, the other
            results are not displayed.
        - in: query
          name: suffix
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Suffix
          required: false
          description: >-
            The model suffix to filter on. When set, the other results are not
            displayed.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/JobsOut'
      description: Get a list of fine-tuning jobs for your organization and user.
      tags:
        - fine-tuning
    post:
      operationId: jobs_api_routes_fine_tuning_create_fine_tuning_job
      summary: Create Fine Tuning Job
      parameters:
        - in: query
          name: dry_run
          schema:
            anyOf:
              - type: boolean
              - type: 'null'
            title: Dry Run
          required: false
          description: >
            * If `true` the job is not spawned, instead the query returns a
            handful of useful metadata
              for the user to perform sanity checks (see `LegacyJobMetadataOut` response).
            * Otherwise, the job is started and the query returns the job ID
            along with some of the
              input parameters (see `JobOut` response).
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                anyOf:
                  - $ref: '#/components/schemas/JobOut'
                  - $ref: '#/components/schemas/LegacyJobMetadataOut'
                title: Response
      description: Create a new fine-tuning job, it will be queued for processing.
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/JobIn'
        required: true
      tags:
        - fine-tuning
  /v1/fine_tuning/jobs/{job_id}:
    get:
      operationId: jobs_api_routes_fine_tuning_get_fine_tuning_job
      summary: Get Fine Tuning Job
      parameters:
        - in: path
          name: job_id
          schema:
            format: uuid
            title: Job Id
            type: string
          required: true
          description: The ID of the job to analyse.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DetailedJobOut'
      description: Get a fine-tuned job details by its UUID.
      tags:
        - fine-tuning
  /v1/fine_tuning/jobs/{job_id}/cancel:
    post:
      operationId: jobs_api_routes_fine_tuning_cancel_fine_tuning_job
      summary: Cancel Fine Tuning Job
      parameters:
        - in: path
          name: job_id
          schema:
            format: uuid
            title: Job Id
            type: string
          required: true
          description: The ID of the job to cancel.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DetailedJobOut'
      description: Request the cancellation of a fine tuning job.
      tags:
        - fine-tuning
  /v1/fine_tuning/jobs/{job_id}/start:
    post:
      operationId: jobs_api_routes_fine_tuning_start_fine_tuning_job
      summary: Start Fine Tuning Job
      parameters:
        - in: path
          name: job_id
          schema:
            format: uuid
            title: Job Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DetailedJobOut'
      description: Request the start of a validated fine tuning job.
      tags:
        - fine-tuning
  /v1/fine_tuning/models/{model_id}:
    patch:
      operationId: jobs_api_routes_fine_tuning_update_fine_tuned_model
      summary: Update Fine Tuned Model
      parameters:
        - in: path
          name: model_id
          schema:
            title: Model Id
            type: string
          required: true
          example: ft:open-mistral-7b:587a6b29:20240514:7e773925
          description: The ID of the model to update.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FTModelOut'
      description: Update a model name or description.
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateFTModelIn'
        required: true
      tags:
        - models
  /v1/fine_tuning/models/{model_id}/archive:
    post:
      operationId: jobs_api_routes_fine_tuning_archive_fine_tuned_model
      summary: Archive Fine Tuned Model
      parameters:
        - in: path
          name: model_id
          schema:
            title: Model Id
            type: string
          required: true
          example: ft:open-mistral-7b:587a6b29:20240514:7e773925
          description: The ID of the model to archive.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ArchiveFTModelOut'
      description: Archive a fine-tuned model.
      tags:
        - models
    delete:
      operationId: jobs_api_routes_fine_tuning_unarchive_fine_tuned_model
      summary: Unarchive Fine Tuned Model
      parameters:
        - in: path
          name: model_id
          schema:
            title: Model Id
            type: string
          required: true
          example: ft:open-mistral-7b:587a6b29:20240514:7e773925
          description: The ID of the model to unarchive.
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UnarchiveFTModelOut'
      description: Un-archive a fine-tuned model.
      tags:
        - models
  /v1/batch/jobs:
    get:
      operationId: jobs_api_routes_batch_get_batch_jobs
      summary: Get Batch Jobs
      parameters:
        - in: query
          name: page
          schema:
            default: 0
            title: Page
            type: integer
          required: false
        - in: query
          name: page_size
          schema:
            default: 100
            title: Page Size
            type: integer
          required: false
        - in: query
          name: model
          schema:
            anyOf:
              - type: string
              - type: 'null'
            title: Model
          required: false
        - in: query
          name: metadata
          schema:
            anyOf:
              - type: object
                additionalProperties: true
              - type: 'null'
            title: Metadata
          required: false
        - in: query
          name: created_after
          schema:
            anyOf:
              - format: date-time
                type: string
              - type: 'null'
            title: Created After
          required: false
        - in: query
          name: created_by_me
          schema:
            default: false
            title: Created By Me
            type: boolean
          required: false
        - in: query
          name: status
          schema:
            anyOf:
              - $ref: '#/components/schemas/BatchJobStatus'
              - type: 'null'
          required: false
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BatchJobsOut'
      description: Get a list of batch jobs for your organization and user.
      tags:
        - batch
    post:
      operationId: jobs_api_routes_batch_create_batch_job
      summary: Create Batch Job
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BatchJobOut'
      description: Create a new batch job, it will be queued for processing.
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BatchJobIn'
        required: true
      tags:
        - batch
  /v1/batch/jobs/{job_id}:
    get:
      operationId: jobs_api_routes_batch_get_batch_job
      summary: Get Batch Job
      parameters:
        - in: path
          name: job_id
          schema:
            format: uuid
            title: Job Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BatchJobOut'
      description: Get a batch job details by its UUID.
      tags:
        - batch
  /v1/batch/jobs/{job_id}/cancel:
    post:
      operationId: jobs_api_routes_batch_cancel_batch_job
      summary: Cancel Batch Job
      parameters:
        - in: path
          name: job_id
          schema:
            format: uuid
            title: Job Id
            type: string
          required: true
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BatchJobOut'
      description: Request the cancellation of a batch job.
      tags:
        - batch
  /v1/chat/completions:
    post:
      summary: Chat Completion
      operationId: chat_completion_v1_chat_completions_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ChatCompletionRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ChatCompletionResponse'
            text/event-stream:
              schema:
                $ref: '#/components/schemas/CompletionEvent'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - chat
  /v1/fim/completions:
    post:
      summary: Fim Completion
      description: FIM completion.
      operationId: fim_completion_v1_fim_completions_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/FIMCompletionRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FIMCompletionResponse'
            text/event-stream:
              schema:
                $ref: '#/components/schemas/CompletionEvent'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - fim
  /v1/agents/completions:
    post:
      summary: Agents Completion
      operationId: agents_completion_v1_agents_completions_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/AgentsCompletionRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ChatCompletionResponse'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - agents
  /v1/embeddings:
    post:
      summary: Embeddings
      description: Embeddings
      operationId: embeddings_v1_embeddings_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EmbeddingRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/EmbeddingResponse'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - embeddings
  /v1/moderations:
    post:
      summary: Moderations
      operationId: moderations_v1_moderations_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ClassificationRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ClassificationResponse'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - classifiers
  /v1/chat/moderations:
    post:
      summary: Moderations Chat
      operationId: moderations_chat_v1_chat_moderations_post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ChatClassificationRequest'
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ClassificationResponse'
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HTTPValidationError'
      tags:
        - classifiers
components:
  schemas:
    BaseModelCard:
      properties:
        id:
          type: string
          title: Id
        object:
          type: string
          title: Object
          default: model
        created:
          type: integer
          title: Created
        owned_by:
          type: string
          title: Owned By
          default: mistralai
        capabilities:
          $ref: '#/components/schemas/ModelCapabilities'
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
        description:
          anyOf:
            - type: string
            - type: 'null'
          title: Description
        max_context_length:
          type: integer
          title: Max Context Length
          default: 32768
        aliases:
          items:
            type: string
          type: array
          title: Aliases
          default: []
        deprecation:
          anyOf:
            - type: string
              format: date-time
            - type: 'null'
          title: Deprecation
        default_model_temperature:
          anyOf:
            - type: number
            - type: 'null'
          title: Default Model Temperature
        type:
          type: string
          enum:
            - base
          const: base
          title: Type
          default: base
      type: object
      required:
        - id
        - capabilities
      title: BaseModelCard
    DeleteModelOut:
      properties:
        id:
          type: string
          title: Id
          description: The ID of the deleted model.
          examples:
            - ft:open-mistral-7b:587a6b29:20240514:7e773925
        object:
          type: string
          title: Object
          default: model
          description: The object type that was deleted
        deleted:
          type: boolean
          title: Deleted
          default: true
          description: The deletion status
          examples:
            - true
      type: object
      required:
        - id
      title: DeleteModelOut
    FTModelCard:
      properties:
        id:
          type: string
          title: Id
        object:
          type: string
          title: Object
          default: model
        created:
          type: integer
          title: Created
        owned_by:
          type: string
          title: Owned By
          default: mistralai
        capabilities:
          $ref: '#/components/schemas/ModelCapabilities'
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
        description:
          anyOf:
            - type: string
            - type: 'null'
          title: Description
        max_context_length:
          type: integer
          title: Max Context Length
          default: 32768
        aliases:
          items:
            type: string
          type: array
          title: Aliases
          default: []
        deprecation:
          anyOf:
            - type: string
              format: date-time
            - type: 'null'
          title: Deprecation
        default_model_temperature:
          anyOf:
            - type: number
            - type: 'null'
          title: Default Model Temperature
        type:
          type: string
          enum:
            - fine-tuned
          const: fine-tuned
          title: Type
          default: fine-tuned
        job:
          type: string
          title: Job
        root:
          type: string
          title: Root
        archived:
          type: boolean
          title: Archived
          default: false
      type: object
      required:
        - id
        - capabilities
        - job
        - root
      title: FTModelCard
      description: Extra fields for fine-tuned models.
    HTTPValidationError:
      properties:
        detail:
          items:
            $ref: '#/components/schemas/ValidationError'
          type: array
          title: Detail
      type: object
      title: HTTPValidationError
    ModelCapabilities:
      properties:
        completion_chat:
          type: boolean
          title: Completion Chat
          default: true
        completion_fim:
          type: boolean
          title: Completion Fim
          default: false
        function_calling:
          type: boolean
          title: Function Calling
          default: true
        fine_tuning:
          type: boolean
          title: Fine Tuning
          default: false
        vision:
          type: boolean
          title: Vision
          default: false
      type: object
      title: ModelCapabilities
    ModelList:
      properties:
        object:
          type: string
          title: Object
          default: list
        data:
          items:
            oneOf:
              - $ref: '#/components/schemas/BaseModelCard'
              - $ref: '#/components/schemas/FTModelCard'
            discriminator:
              propertyName: type
              mapping:
                base: '#/components/schemas/BaseModelCard'
                fine-tuned: '#/components/schemas/FTModelCard'
          type: array
          title: Data
      type: object
      title: ModelList
    ValidationError:
      properties:
        loc:
          items:
            anyOf:
              - type: string
              - type: integer
          type: array
          title: Location
        msg:
          type: string
          title: Message
        type:
          type: string
          title: Error Type
      type: object
      required:
        - loc
        - msg
        - type
      title: ValidationError
    FilePurpose:
      title: FilePurpose
      type: string
      enum:
        - fine-tune
        - batch
    SampleType:
      title: SampleType
      type: string
      enum:
        - pretrain
        - instruct
        - batch_request
        - batch_result
        - batch_error
    Source:
      enum:
        - upload
        - repository
        - mistral
      title: Source
      type: string
    UploadFileOut:
      properties:
        id:
          format: uuid
          title: Id
          type: string
          description: The unique identifier of the file.
          examples:
            - 497f6eca-6276-4993-bfeb-53cbbbba6f09
        object:
          title: Object
          type: string
          description: The object type, which is always "file".
          examples:
            - file
        bytes:
          title: Bytes
          type: integer
          description: The size of the file, in bytes.
          examples:
            - 13000
        created_at:
          title: Created At
          type: integer
          description: The UNIX timestamp (in seconds) of the event.
          examples:
            - 1716963433
        filename:
          title: Filename
          type: string
          description: The name of the uploaded file.
          examples:
            - files_upload.jsonl
        purpose:
          $ref: '#/components/schemas/FilePurpose'
          description: >-
            The intended purpose of the uploaded file. Only accepts fine-tuning
            (`fine-tune`) for now.
          examples:
            - fine-tune
        sample_type:
          $ref: '#/components/schemas/SampleType'
        num_lines:
          anyOf:
            - type: integer
            - type: 'null'
          title: Num Lines
        source:
          $ref: '#/components/schemas/Source'
      required:
        - id
        - object
        - bytes
        - created_at
        - filename
        - purpose
        - sample_type
        - source
      title: UploadFileOut
      type: object
    FileSchema:
      properties:
        id:
          format: uuid
          title: Id
          type: string
          description: The unique identifier of the file.
          examples:
            - 497f6eca-6276-4993-bfeb-53cbbbba6f09
        object:
          title: Object
          type: string
          description: The object type, which is always "file".
          examples:
            - file
        bytes:
          title: Bytes
          type: integer
          description: The size of the file, in bytes.
          examples:
            - 13000
        created_at:
          title: Created At
          type: integer
          description: The UNIX timestamp (in seconds) of the event.
          examples:
            - 1716963433
        filename:
          title: Filename
          type: string
          description: The name of the uploaded file.
          examples:
            - files_upload.jsonl
        purpose:
          $ref: '#/components/schemas/FilePurpose'
          description: >-
            The intended purpose of the uploaded file. Only accepts fine-tuning
            (`fine-tune`) for now.
          examples:
            - fine-tune
        sample_type:
          $ref: '#/components/schemas/SampleType'
        num_lines:
          anyOf:
            - type: integer
            - type: 'null'
          title: Num Lines
        source:
          $ref: '#/components/schemas/Source'
      required:
        - id
        - object
        - bytes
        - created_at
        - filename
        - purpose
        - sample_type
        - source
      title: FileSchema
      type: object
    ListFilesOut:
      properties:
        data:
          items:
            $ref: '#/components/schemas/FileSchema'
          title: Data
          type: array
        object:
          title: Object
          type: string
        total:
          title: Total
          type: integer
      required:
        - data
        - object
        - total
      title: ListFilesOut
      type: object
    RetrieveFileOut:
      properties:
        id:
          format: uuid
          title: Id
          type: string
          description: The unique identifier of the file.
          examples:
            - 497f6eca-6276-4993-bfeb-53cbbbba6f09
        object:
          title: Object
          type: string
          description: The object type, which is always "file".
          examples:
            - file
        bytes:
          title: Bytes
          type: integer
          description: The size of the file, in bytes.
          examples:
            - 13000
        created_at:
          title: Created At
          type: integer
          description: The UNIX timestamp (in seconds) of the event.
          examples:
            - 1716963433
        filename:
          title: Filename
          type: string
          description: The name of the uploaded file.
          examples:
            - files_upload.jsonl
        purpose:
          $ref: '#/components/schemas/FilePurpose'
          description: >-
            The intended purpose of the uploaded file. Only accepts fine-tuning
            (`fine-tune`) for now.
          examples:
            - fine-tune
        sample_type:
          $ref: '#/components/schemas/SampleType'
        num_lines:
          anyOf:
            - type: integer
            - type: 'null'
          title: Num Lines
        source:
          $ref: '#/components/schemas/Source'
        deleted:
          title: Deleted
          type: boolean
      required:
        - id
        - object
        - bytes
        - created_at
        - filename
        - purpose
        - sample_type
        - source
        - deleted
      title: RetrieveFileOut
      type: object
    DeleteFileOut:
      properties:
        id:
          format: uuid
          title: Id
          type: string
          description: The ID of the deleted file.
          examples:
            - 497f6eca-6276-4993-bfeb-53cbbbba6f09
        object:
          title: Object
          type: string
          description: The object type that was deleted
          examples:
            - file
        deleted:
          title: Deleted
          type: boolean
          description: The deletion status.
          examples:
            - false
      required:
        - id
        - object
        - deleted
      title: DeleteFileOut
      type: object
    FileSignedURL:
      properties:
        url:
          title: Url
          type: string
      required:
        - url
      title: FileSignedURL
      type: object
    FineTuneableModel:
      enum:
        - open-mistral-7b
        - mistral-small-latest
        - codestral-latest
        - mistral-large-latest
        - open-mistral-nemo
        - ministral-3b-latest
      title: FineTuneableModel
      type: string
      description: The name of the model to fine-tune.
    GithubRepositoryOut:
      properties:
        type:
          const: github
          default: github
          enum:
            - github
          title: Type
          type: string
        name:
          title: Name
          type: string
        owner:
          title: Owner
          type: string
        ref:
          anyOf:
            - type: string
            - type: 'null'
          title: Ref
        weight:
          default: 1
          exclusiveMinimum: 0
          title: Weight
          type: number
        commit_id:
          maxLength: 40
          minLength: 40
          title: Commit Id
          type: string
      required:
        - name
        - owner
        - commit_id
      title: GithubRepositoryOut
      type: object
    JobMetadataOut:
      properties:
        expected_duration_seconds:
          anyOf:
            - type: integer
            - type: 'null'
          title: Expected Duration Seconds
        cost:
          anyOf:
            - type: number
            - type: 'null'
          title: Cost
        cost_currency:
          anyOf:
            - type: string
            - type: 'null'
          title: Cost Currency
        train_tokens_per_step:
          anyOf:
            - type: integer
            - type: 'null'
          title: Train Tokens Per Step
        train_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Train Tokens
        data_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Data Tokens
        estimated_start_time:
          anyOf:
            - type: integer
            - type: 'null'
          title: Estimated Start Time
      title: JobMetadataOut
      type: object
    JobOut:
      properties:
        id:
          format: uuid
          title: Id
          type: string
          description: The ID of the job.
        auto_start:
          title: Auto Start
          type: boolean
        hyperparameters:
          $ref: '#/components/schemas/TrainingParameters'
        model:
          $ref: '#/components/schemas/FineTuneableModel'
        status:
          enum:
            - QUEUED
            - STARTED
            - VALIDATING
            - VALIDATED
            - RUNNING
            - FAILED_VALIDATION
            - FAILED
            - SUCCESS
            - CANCELLED
            - CANCELLATION_REQUESTED
          title: Status
          type: string
          description: The current status of the fine-tuning job.
        job_type:
          title: Job Type
          type: string
          description: The type of job (`FT` for fine-tuning).
        created_at:
          title: Created At
          type: integer
          description: >-
            The UNIX timestamp (in seconds) for when the fine-tuning job was
            created.
        modified_at:
          title: Modified At
          type: integer
          description: >-
            The UNIX timestamp (in seconds) for when the fine-tuning job was
            last modified.
        training_files:
          items:
            format: uuid
            type: string
          title: Training Files
          type: array
          description: >-
            A list containing the IDs of uploaded files that contain training
            data.
        validation_files:
          anyOf:
            - items:
                format: uuid
                type: string
              type: array
            - type: 'null'
          default: []
          title: Validation Files
          description: >-
            A list containing the IDs of uploaded files that contain validation
            data.
        object:
          const: job
          default: job
          enum:
            - job
          title: Object
          type: string
          description: The object type of the fine-tuning job.
        fine_tuned_model:
          anyOf:
            - type: string
            - type: 'null'
          title: Fine Tuned Model
          description: >-
            The name of the fine-tuned model that is being created. The value
            will be `null` if the fine-tuning job is still running.
        suffix:
          anyOf:
            - type: string
            - type: 'null'
          title: Suffix
          description: >-
            Optional text/code that adds more context for the model. When given
            a `prompt` and a `suffix` the model will fill what is between them.
            When `suffix` is not provided, the model will simply execute
            completion starting with `prompt`.
        integrations:
          anyOf:
            - items:
                discriminator:
                  mapping:
                    wandb: '#/components/schemas/WandbIntegrationOut'
                  propertyName: type
                oneOf:
                  - $ref: '#/components/schemas/WandbIntegrationOut'
              type: array
            - type: 'null'
          title: Integrations
          description: A list of integrations enabled for your fine-tuning job.
        trained_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Trained Tokens
          description: Total number of tokens trained.
        repositories:
          default: []
          items:
            discriminator:
              mapping:
                github: '#/components/schemas/GithubRepositoryOut'
              propertyName: type
            oneOf:
              - $ref: '#/components/schemas/GithubRepositoryOut'
          title: Repositories
          type: array
        metadata:
          anyOf:
            - $ref: '#/components/schemas/JobMetadataOut'
            - type: 'null'
      required:
        - id
        - auto_start
        - hyperparameters
        - model
        - status
        - job_type
        - created_at
        - modified_at
        - training_files
      title: JobOut
      type: object
    JobsOut:
      properties:
        data:
          default: []
          items:
            $ref: '#/components/schemas/JobOut'
          title: Data
          type: array
        object:
          const: list
          default: list
          enum:
            - list
          title: Object
          type: string
        total:
          title: Total
          type: integer
      required:
        - total
      title: JobsOut
      type: object
    TrainingParameters:
      properties:
        training_steps:
          anyOf:
            - minimum: 1
              type: integer
            - type: 'null'
          title: Training Steps
        learning_rate:
          default: 0.0001
          maximum: 1
          minimum: 1.e-8
          title: Learning Rate
          type: number
        weight_decay:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.1
          title: Weight Decay
        warmup_fraction:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.05
          title: Warmup Fraction
        epochs:
          anyOf:
            - exclusiveMinimum: 0
              type: number
            - type: 'null'
          title: Epochs
        fim_ratio:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.9
          title: Fim Ratio
        seq_len:
          anyOf:
            - minimum: 100
              type: integer
            - type: 'null'
          title: Seq Len
      title: TrainingParameters
      type: object
    WandbIntegrationOut:
      properties:
        type:
          const: wandb
          default: wandb
          enum:
            - wandb
          title: Type
          type: string
        project:
          title: Project
          type: string
          description: The name of the project that the new run will be created under.
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
          description: >-
            A display name to set for the run. If not set, will use the job ID
            as the name.
        run_name:
          anyOf:
            - type: string
            - type: 'null'
          title: Run Name
      required:
        - project
      title: WandbIntegrationOut
      type: object
    LegacyJobMetadataOut:
      properties:
        expected_duration_seconds:
          anyOf:
            - type: integer
            - type: 'null'
          title: Expected Duration Seconds
          description: >-
            The approximated time (in seconds) for the fine-tuning process to
            complete.
          examples:
            - 220
        cost:
          anyOf:
            - type: number
            - type: 'null'
          title: Cost
          description: The cost of the fine-tuning job.
          examples:
            - 10
        cost_currency:
          anyOf:
            - type: string
            - type: 'null'
          title: Cost Currency
          description: The currency used for the fine-tuning job cost.
          examples:
            - EUR
        train_tokens_per_step:
          anyOf:
            - type: integer
            - type: 'null'
          title: Train Tokens Per Step
          description: The number of tokens consumed by one training step.
          examples:
            - 131072
        train_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Train Tokens
          description: The total number of tokens used during the fine-tuning process.
          examples:
            - 1310720
        data_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Data Tokens
          description: The total number of tokens in the training dataset.
          examples:
            - 305375
        estimated_start_time:
          anyOf:
            - type: integer
            - type: 'null'
          title: Estimated Start Time
        deprecated:
          default: true
          title: Deprecated
          type: boolean
        details:
          title: Details
          type: string
        epochs:
          anyOf:
            - type: number
            - type: 'null'
          title: Epochs
          description: The number of complete passes through the entire training dataset.
          examples:
            - 4.2922
        training_steps:
          anyOf:
            - type: integer
            - type: 'null'
          title: Training Steps
          description: >-
            The number of training steps to perform. A training step refers to a
            single update of the model weights during the fine-tuning process.
            This update is typically calculated using a batch of samples from
            the training dataset.
          examples:
            - 10
        object:
          const: job.metadata
          default: job.metadata
          enum:
            - job.metadata
          title: Object
          type: string
      required:
        - details
      title: LegacyJobMetadataOut
      type: object
    GithubRepositoryIn:
      properties:
        type:
          const: github
          default: github
          enum:
            - github
          title: Type
          type: string
        name:
          title: Name
          type: string
        owner:
          title: Owner
          type: string
        ref:
          anyOf:
            - type: string
            - type: 'null'
          title: Ref
        weight:
          default: 1
          exclusiveMinimum: 0
          title: Weight
          type: number
        token:
          title: Token
          type: string
      required:
        - name
        - owner
        - token
      title: GithubRepositoryIn
      type: object
    JobIn:
      properties:
        model:
          $ref: '#/components/schemas/FineTuneableModel'
        training_files:
          default: []
          items:
            $ref: '#/components/schemas/TrainingFile'
          title: Training Files
          type: array
        validation_files:
          anyOf:
            - items:
                format: uuid
                type: string
              type: array
            - type: 'null'
          title: Validation Files
          description: >-
            A list containing the IDs of uploaded files that contain validation
            data. If you provide these files, the data is used to generate
            validation metrics periodically during fine-tuning. These metrics
            can be viewed in `checkpoints` when getting the status of a running
            fine-tuning job. The same data should not be present in both train
            and validation files.
        hyperparameters:
          $ref: '#/components/schemas/TrainingParametersIn'
        suffix:
          anyOf:
            - maxLength: 18
              type: string
            - type: 'null'
          title: Suffix
          description: >-
            A string that will be added to your fine-tuning model name. For
            example, a suffix of "my-great-model" would produce a model name
            like `ft:open-mistral-7b:my-great-model:xxx...`
        integrations:
          anyOf:
            - items:
                discriminator:
                  mapping:
                    wandb: '#/components/schemas/WandbIntegration'
                  propertyName: type
                oneOf:
                  - $ref: '#/components/schemas/WandbIntegration'
              type: array
            - type: 'null'
          title: Integrations
          description: A list of integrations to enable for your fine-tuning job.
        repositories:
          default: []
          items:
            discriminator:
              mapping:
                github: '#/components/schemas/GithubRepositoryIn'
              propertyName: type
            oneOf:
              - $ref: '#/components/schemas/GithubRepositoryIn'
          maxItems: 50
          title: Repositories
          type: array
        auto_start:
          description: This field will be required in a future release.
          title: Auto Start
          type: boolean
      required:
        - model
        - hyperparameters
      title: JobIn
      type: object
    TrainingFile:
      properties:
        file_id:
          format: uuid
          title: File Id
          type: string
        weight:
          default: 1
          exclusiveMinimum: 0
          title: Weight
          type: number
      required:
        - file_id
      title: TrainingFile
      type: object
    TrainingParametersIn:
      properties:
        training_steps:
          anyOf:
            - minimum: 1
              type: integer
            - type: 'null'
          title: Training Steps
          description: >-
            The number of training steps to perform. A training step refers to a
            single update of the model weights during the fine-tuning process.
            This update is typically calculated using a batch of samples from
            the training dataset.
        learning_rate:
          default: 0.0001
          maximum: 1
          minimum: 1.e-8
          title: Learning Rate
          type: number
          description: >-
            A parameter describing how much to adjust the pre-trained model's
            weights in response to the estimated error each time the weights are
            updated during the fine-tuning process.
        weight_decay:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.1
          title: Weight Decay
          description: >-
            (Advanced Usage) Weight decay adds a term to the loss function that
            is proportional to the sum of the squared weights. This term reduces
            the magnitude of the weights and prevents them from growing too
            large.
        warmup_fraction:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.05
          title: Warmup Fraction
          description: >-
            (Advanced Usage) A parameter that specifies the percentage of the
            total training steps at which the learning rate warm-up phase ends.
            During this phase, the learning rate gradually increases from a
            small value to the initial learning rate, helping to stabilize the
            training process and improve convergence. Similar to `pct_start` in
            [mistral-finetune](https://github.com/mistralai/mistral-finetune)
        epochs:
          anyOf:
            - exclusiveMinimum: 0
              type: number
            - type: 'null'
          title: Epochs
        fim_ratio:
          anyOf:
            - maximum: 1
              minimum: 0
              type: number
            - type: 'null'
          default: 0.9
          title: Fim Ratio
        seq_len:
          anyOf:
            - minimum: 100
              type: integer
            - type: 'null'
          title: Seq Len
      title: TrainingParametersIn
      type: object
      description: The fine-tuning hyperparameter settings used in a fine-tune job.
    WandbIntegration:
      properties:
        type:
          const: wandb
          default: wandb
          enum:
            - wandb
          title: Type
          type: string
        project:
          title: Project
          type: string
          description: The name of the project that the new run will be created under.
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
          description: >-
            A display name to set for the run. If not set, will use the job ID
            as the name.
        api_key:
          maxLength: 40
          minLength: 40
          title: Api Key
          type: string
          description: The WandB API key to use for authentication.
        run_name:
          anyOf:
            - type: string
            - type: 'null'
          title: Run Name
      required:
        - project
        - api_key
      title: WandbIntegration
      type: object
    CheckpointOut:
      properties:
        metrics:
          $ref: '#/components/schemas/MetricOut'
        step_number:
          title: Step Number
          type: integer
          description: The step number that the checkpoint was created at.
        created_at:
          title: Created At
          type: integer
          description: The UNIX timestamp (in seconds) for when the checkpoint was created.
          examples:
            - 1716963433
      required:
        - metrics
        - step_number
        - created_at
      title: CheckpointOut
      type: object
    DetailedJobOut:
      properties:
        id:
          format: uuid
          title: Id
          type: string
        auto_start:
          title: Auto Start
          type: boolean
        hyperparameters:
          $ref: '#/components/schemas/TrainingParameters'
        model:
          $ref: '#/components/schemas/FineTuneableModel'
        status:
          enum:
            - QUEUED
            - STARTED
            - VALIDATING
            - VALIDATED
            - RUNNING
            - FAILED_VALIDATION
            - FAILED
            - SUCCESS
            - CANCELLED
            - CANCELLATION_REQUESTED
          title: Status
          type: string
        job_type:
          title: Job Type
          type: string
        created_at:
          title: Created At
          type: integer
        modified_at:
          title: Modified At
          type: integer
        training_files:
          items:
            format: uuid
            type: string
          title: Training Files
          type: array
        validation_files:
          anyOf:
            - items:
                format: uuid
                type: string
              type: array
            - type: 'null'
          default: []
          title: Validation Files
        object:
          const: job
          default: job
          enum:
            - job
          title: Object
          type: string
        fine_tuned_model:
          anyOf:
            - type: string
            - type: 'null'
          title: Fine Tuned Model
        suffix:
          anyOf:
            - type: string
            - type: 'null'
          title: Suffix
        integrations:
          anyOf:
            - items:
                discriminator:
                  mapping:
                    wandb: '#/components/schemas/WandbIntegrationOut'
                  propertyName: type
                oneOf:
                  - $ref: '#/components/schemas/WandbIntegrationOut'
              type: array
            - type: 'null'
          title: Integrations
        trained_tokens:
          anyOf:
            - type: integer
            - type: 'null'
          title: Trained Tokens
        repositories:
          default: []
          items:
            discriminator:
              mapping:
                github: '#/components/schemas/GithubRepositoryOut'
              propertyName: type
            oneOf:
              - $ref: '#/components/schemas/GithubRepositoryOut'
          title: Repositories
          type: array
        metadata:
          anyOf:
            - $ref: '#/components/schemas/JobMetadataOut'
            - type: 'null'
        events:
          default: []
          items:
            $ref: '#/components/schemas/EventOut'
          title: Events
          type: array
          description: >-
            Event items are created every time the status of a fine-tuning job
            changes. The timestamped list of all events is accessible here.
        checkpoints:
          default: []
          items:
            $ref: '#/components/schemas/CheckpointOut'
          title: Checkpoints
          type: array
      required:
        - id
        - auto_start
        - hyperparameters
        - model
        - status
        - job_type
        - created_at
        - modified_at
        - training_files
      title: DetailedJobOut
      type: object
    EventOut:
      properties:
        name:
          title: Name
          type: string
          description: The name of the event.
        data:
          anyOf:
            - type: object
              additionalProperties: true
            - type: 'null'
          title: Data
        created_at:
          title: Created At
          type: integer
          description: The UNIX timestamp (in seconds) of the event.
      required:
        - name
        - created_at
      title: EventOut
      type: object
    MetricOut:
      properties:
        train_loss:
          anyOf:
            - type: number
            - type: 'null'
          title: Train Loss
        valid_loss:
          anyOf:
            - type: number
            - type: 'null'
          title: Valid Loss
        valid_mean_token_accuracy:
          anyOf:
            - type: number
            - type: 'null'
          title: Valid Mean Token Accuracy
      title: MetricOut
      type: object
      description: >-
        Metrics at the step number during the fine-tuning job. Use these metrics
        to assess if the training is going smoothly (loss should decrease, token
        accuracy should increase).
    FTModelCapabilitiesOut:
      properties:
        completion_chat:
          default: true
          title: Completion Chat
          type: boolean
        completion_fim:
          default: false
          title: Completion Fim
          type: boolean
        function_calling:
          default: false
          title: Function Calling
          type: boolean
        fine_tuning:
          default: false
          title: Fine Tuning
          type: boolean
      title: FTModelCapabilitiesOut
      type: object
    FTModelOut:
      properties:
        id:
          title: Id
          type: string
        object:
          const: model
          default: model
          enum:
            - model
          title: Object
          type: string
        created:
          title: Created
          type: integer
        owned_by:
          title: Owned By
          type: string
        root:
          title: Root
          type: string
        archived:
          title: Archived
          type: boolean
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
        description:
          anyOf:
            - type: string
            - type: 'null'
          title: Description
        capabilities:
          $ref: '#/components/schemas/FTModelCapabilitiesOut'
        max_context_length:
          default: 32768
          title: Max Context Length
          type: integer
        aliases:
          default: []
          items:
            type: string
          title: Aliases
          type: array
        job:
          format: uuid
          title: Job
          type: string
      required:
        - id
        - created
        - owned_by
        - root
        - archived
        - capabilities
        - job
      title: FTModelOut
      type: object
    UpdateFTModelIn:
      properties:
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
        description:
          anyOf:
            - type: string
            - type: 'null'
          title: Description
      title: UpdateFTModelIn
      type: object
    ArchiveFTModelOut:
      properties:
        id:
          title: Id
          type: string
        object:
          const: model
          default: model
          enum:
            - model
          title: Object
          type: string
        archived:
          default: true
          title: Archived
          type: boolean
      required:
        - id
      title: ArchiveFTModelOut
      type: object
    UnarchiveFTModelOut:
      properties:
        id:
          title: Id
          type: string
        object:
          const: model
          default: model
          enum:
            - model
          title: Object
          type: string
        archived:
          default: false
          title: Archived
          type: boolean
      required:
        - id
      title: UnarchiveFTModelOut
      type: object
    BatchJobStatus:
      enum:
        - QUEUED
        - RUNNING
        - SUCCESS
        - FAILED
        - TIMEOUT_EXCEEDED
        - CANCELLATION_REQUESTED
        - CANCELLED
      title: BatchJobStatus
      type: string
    BatchError:
      properties:
        message:
          title: Message
          type: string
        count:
          default: 1
          title: Count
          type: integer
      required:
        - message
      title: BatchError
      type: object
    BatchJobOut:
      properties:
        id:
          title: Id
          type: string
        object:
          const: batch
          default: batch
          enum:
            - batch
          title: Object
          type: string
        input_files:
          items:
            format: uuid
            type: string
          title: Input Files
          type: array
        metadata:
          anyOf:
            - type: object
              additionalProperties: true
            - type: 'null'
          title: Metadata
        endpoint:
          title: Endpoint
          type: string
        model:
          title: Model
          type: string
        output_file:
          anyOf:
            - format: uuid
              type: string
            - type: 'null'
          title: Output File
        error_file:
          anyOf:
            - format: uuid
              type: string
            - type: 'null'
          title: Error File
        errors:
          items:
            $ref: '#/components/schemas/BatchError'
          title: Errors
          type: array
        status:
          $ref: '#/components/schemas/BatchJobStatus'
        created_at:
          title: Created At
          type: integer
        total_requests:
          title: Total Requests
          type: integer
        completed_requests:
          title: Completed Requests
          type: integer
        succeeded_requests:
          title: Succeeded Requests
          type: integer
        failed_requests:
          title: Failed Requests
          type: integer
        started_at:
          anyOf:
            - type: integer
            - type: 'null'
          title: Started At
        completed_at:
          anyOf:
            - type: integer
            - type: 'null'
          title: Completed At
      required:
        - id
        - input_files
        - endpoint
        - model
        - errors
        - status
        - created_at
        - total_requests
        - completed_requests
        - succeeded_requests
        - failed_requests
      title: BatchJobOut
      type: object
    BatchJobsOut:
      properties:
        data:
          default: []
          items:
            $ref: '#/components/schemas/BatchJobOut'
          title: Data
          type: array
        object:
          const: list
          default: list
          enum:
            - list
          title: Object
          type: string
        total:
          title: Total
          type: integer
      required:
        - total
      title: BatchJobsOut
      type: object
    ApiEndpoint:
      title: ApiEndpoint
      type: string
      enum:
        - /v1/chat/completions
        - /v1/embeddings
        - /v1/fim/completions
        - /v1/moderations
        - /v1/chat/moderations
    BatchJobIn:
      properties:
        input_files:
          items:
            format: uuid
            type: string
          title: Input Files
          type: array
        endpoint:
          $ref: '#/components/schemas/ApiEndpoint'
        model:
          title: Model
          type: string
        metadata:
          anyOf:
            - additionalProperties:
                maxLength: 512
                minLength: 1
                type: string
              type: object
            - type: 'null'
          title: Metadata
        timeout_hours:
          default: 24
          title: Timeout Hours
          type: integer
      required:
        - input_files
        - endpoint
        - model
      title: BatchJobIn
      type: object
    AssistantMessage:
      properties:
        content:
          title: Content
          anyOf:
            - type: string
            - type: 'null'
            - items:
                $ref: '#/components/schemas/ContentChunk'
              type: array
        tool_calls:
          anyOf:
            - items:
                $ref: '#/components/schemas/ToolCall'
              type: array
            - type: 'null'
          title: Tool Calls
        prefix:
          type: boolean
          title: Prefix
          default: false
        role:
          type: string
          default: assistant
          title: Role
          enum:
            - assistant
      additionalProperties: false
      type: object
      title: AssistantMessage
    ChatCompletionRequest:
      properties:
        model:
          anyOf:
            - type: string
            - type: 'null'
          title: Model
          description: >-
            ID of the model to use. You can use the [List Available
            Models](/api/#tag/models/operation/list_models_v1_models_get) API to
            see all of your available models, or see our [Model
            overview](/models) for model descriptions.
          examples:
            - mistral-small-latest
        temperature:
          anyOf:
            - type: number
              maximum: 1.5
              minimum: 0
            - type: 'null'
          title: Temperature
          description: >-
            What sampling temperature to use, we recommend between 0.0 and 0.7.
            Higher values like 0.7 will make the output more random, while lower
            values like 0.2 will make it more focused and deterministic. We
            generally recommend altering this or `top_p` but not both. The
            default value varies depending on the model you are targeting. Call
            the `/models` endpoint to retrieve the appropriate value.
        top_p:
          type: number
          maximum: 1
          minimum: 0
          title: Top P
          default: 1
          description: >-
            Nucleus sampling, where the model considers the results of the
            tokens with `top_p` probability mass. So 0.1 means only the tokens
            comprising the top 10% probability mass are considered. We generally
            recommend altering this or `temperature` but not both.
        max_tokens:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Max Tokens
          description: >-
            The maximum number of tokens to generate in the completion. The
            token count of your prompt plus `max_tokens` cannot exceed the
            model's context length.
        stream:
          type: boolean
          title: Stream
          default: false
          description: >-
            Whether to stream back partial progress. If set, tokens will be sent
            as data-only server-side events as they become available, with the
            stream terminated by a data: [DONE] message. Otherwise, the server
            will hold the request open until the timeout or until completion,
            with the response containing the full result as JSON.
        stop:
          anyOf:
            - type: string
            - items:
                type: string
              type: array
          title: Stop
          description: >-
            Stop generation if this token is detected. Or if one of these tokens
            is detected when providing an array
        random_seed:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Random Seed
          description: >-
            The seed to use for random sampling. If set, different calls will
            generate deterministic results.
        messages:
          items:
            oneOf:
              - $ref: '#/components/schemas/SystemMessage'
              - $ref: '#/components/schemas/UserMessage'
              - $ref: '#/components/schemas/AssistantMessage'
              - $ref: '#/components/schemas/ToolMessage'
            discriminator:
              propertyName: role
              mapping:
                assistant: '#/components/schemas/AssistantMessage'
                system: '#/components/schemas/SystemMessage'
                tool: '#/components/schemas/ToolMessage'
                user: '#/components/schemas/UserMessage'
          type: array
          title: Messages
          description: >-
            The prompt(s) to generate completions for, encoded as a list of dict
            with role and content.
          examples:
            - - role: user
                content: Who is the best French painter? Answer in one short sentence.
        response_format:
          $ref: '#/components/schemas/ResponseFormat'
        tools:
          anyOf:
            - items:
                $ref: '#/components/schemas/Tool'
              type: array
            - type: 'null'
          title: Tools
        tool_choice:
          anyOf:
            - $ref: '#/components/schemas/ToolChoice'
            - $ref: '#/components/schemas/ToolChoiceEnum'
          title: Tool Choice
          default: auto
        presence_penalty:
          type: number
          maximum: 2
          minimum: -2
          title: Presence Penalty
          default: 0
          description: >-
            presence_penalty determines how much the model penalizes the
            repetition of words or phrases. A higher presence penalty encourages
            the model to use a wider variety of words and phrases, making the
            output more diverse and creative.
        frequency_penalty:
          type: number
          maximum: 2
          minimum: -2
          title: Frequency Penalty
          default: 0
          description: >-
            frequency_penalty penalizes the repetition of words based on their
            frequency in the generated text. A higher frequency penalty
            discourages the model from repeating words that have already
            appeared frequently in the output, promoting diversity and reducing
            repetition.
        'n':
          anyOf:
            - type: integer
              minimum: 1
            - type: 'null'
          title: 'N'
          description: >-
            Number of completions to return for each request, input tokens are
            only billed once.
        prediction:
          $ref: '#/components/schemas/Prediction'
          default:
            type: content
            content: ''
          description: >-
            Enable users to specify expected results, optimizing response times
            by leveraging known or predictable content. This approach is
            especially effective for updating text documents or code files with
            minimal changes, reducing latency while maintaining high-quality
            results.
        safe_prompt:
          type: boolean
          description: Whether to inject a safety prompt before all conversations.
          default: false
      additionalProperties: false
      type: object
      required:
        - messages
        - model
      title: ChatCompletionRequest
    FIMCompletionRequest:
      properties:
        model:
          anyOf:
            - type: string
            - type: 'null'
          title: Model
          default: codestral-2405
          description: |-
            ID of the model to use. Only compatible for now with:
              - `codestral-2405`
              - `codestral-latest`
          examples:
            - codestral-2405
        temperature:
          anyOf:
            - type: number
              maximum: 1.5
              minimum: 0
            - type: 'null'
          title: Temperature
          description: >-
            What sampling temperature to use, we recommend between 0.0 and 0.7.
            Higher values like 0.7 will make the output more random, while lower
            values like 0.2 will make it more focused and deterministic. We
            generally recommend altering this or `top_p` but not both. The
            default value varies depending on the model you are targeting. Call
            the `/models` endpoint to retrieve the appropriate value.
        top_p:
          type: number
          maximum: 1
          minimum: 0
          title: Top P
          default: 1
          description: >-
            Nucleus sampling, where the model considers the results of the
            tokens with `top_p` probability mass. So 0.1 means only the tokens
            comprising the top 10% probability mass are considered. We generally
            recommend altering this or `temperature` but not both.
        max_tokens:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Max Tokens
          description: >-
            The maximum number of tokens to generate in the completion. The
            token count of your prompt plus `max_tokens` cannot exceed the
            model's context length.
        stream:
          type: boolean
          title: Stream
          default: false
          description: >-
            Whether to stream back partial progress. If set, tokens will be sent
            as data-only server-side events as they become available, with the
            stream terminated by a data: [DONE] message. Otherwise, the server
            will hold the request open until the timeout or until completion,
            with the response containing the full result as JSON.
        stop:
          anyOf:
            - type: string
            - items:
                type: string
              type: array
          title: Stop
          description: >-
            Stop generation if this token is detected. Or if one of these tokens
            is detected when providing an array
        random_seed:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Random Seed
          description: >-
            The seed to use for random sampling. If set, different calls will
            generate deterministic results.
        prompt:
          type: string
          title: Prompt
          description: The text/code to complete.
          examples:
            - def
        suffix:
          anyOf:
            - type: string
            - type: 'null'
          title: Suffix
          default: ''
          description: >-
            Optional text/code that adds more context for the model. When given
            a `prompt` and a `suffix` the model will fill what is between them.
            When `suffix` is not provided, the model will simply execute
            completion starting with `prompt`.
          examples:
            - return a+b
        min_tokens:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Min Tokens
          description: The minimum number of tokens to generate in the completion.
      additionalProperties: false
      type: object
      required:
        - prompt
        - model
      title: FIMCompletionRequest
    Function:
      properties:
        name:
          type: string
          title: Name
        description:
          type: string
          title: Description
          default: ''
        strict:
          type: boolean
          title: Strict
          default: false
        parameters:
          type: object
          title: Parameters
          additionalProperties: true
      additionalProperties: false
      type: object
      required:
        - name
        - parameters
      title: Function
    FunctionCall:
      properties:
        name:
          type: string
          title: Name
        arguments:
          title: Arguments
          anyOf:
            - type: object
              additionalProperties: true
            - type: string
      additionalProperties: false
      type: object
      required:
        - name
        - arguments
      title: FunctionCall
    FunctionName:
      properties:
        name:
          type: string
          title: Name
      additionalProperties: false
      type: object
      required:
        - name
      title: FunctionName
      description: >-
        this restriction of `Function` is used to select a specific function to
        call
    ImageURL:
      properties:
        url:
          type: string
          title: Url
        detail:
          anyOf:
            - type: string
            - type: 'null'
          title: Detail
      additionalProperties: false
      type: object
      required:
        - url
      title: ImageURL
    ImageURLChunk:
      properties:
        image_url:
          anyOf:
            - $ref: '#/components/schemas/ImageURL'
            - type: string
          title: Image Url
        type:
          type: string
          enum:
            - image_url
          title: Type
          default: image_url
      additionalProperties: false
      type: object
      required:
        - image_url
      title: ImageURLChunk
      description: '{"type":"image_url","image_url":{"url":"data:image/png;base64,iVBORw0'
    Prediction:
      properties:
        type:
          type: string
          enum:
            - content
          const: content
          title: Type
          default: content
        content:
          type: string
          title: Content
          default: ''
      additionalProperties: false
      type: object
      title: Prediction
    ReferenceChunk:
      properties:
        reference_ids:
          items:
            type: integer
          type: array
          title: Reference Ids
        type:
          type: string
          enum:
            - reference
          title: Type
          default: reference
      additionalProperties: false
      type: object
      required:
        - reference_ids
      title: ReferenceChunk
    ResponseFormat:
      properties:
        type:
          $ref: '#/components/schemas/ResponseFormats'
          default: text
      additionalProperties: false
      type: object
      title: ResponseFormat
    ResponseFormats:
      type: string
      title: ResponseFormats
      description: >-
        An object specifying the format that the model must output. Setting to
        `{ "type": "json_object" }` enables JSON mode, which guarantees the
        message the model generates is in JSON. When using JSON mode you MUST
        also instruct the model to produce JSON yourself with a system or a user
        message.
      enum:
        - text
        - json_object
    SystemMessage:
      properties:
        content:
          anyOf:
            - type: string
            - items:
                $ref: '#/components/schemas/TextChunk'
              type: array
          title: Content
        role:
          type: string
          default: system
          enum:
            - system
      additionalProperties: false
      type: object
      required:
        - content
      title: SystemMessage
    TextChunk:
      properties:
        text:
          type: string
          title: Text
        type:
          type: string
          enum:
            - text
          title: Type
          default: text
      additionalProperties: false
      type: object
      required:
        - text
      title: TextChunk
    Tool:
      properties:
        type:
          $ref: '#/components/schemas/ToolTypes'
          default: function
        function:
          $ref: '#/components/schemas/Function'
      additionalProperties: false
      type: object
      required:
        - function
      title: Tool
    ToolCall:
      properties:
        id:
          type: string
          title: Id
          default: 'null'
        type:
          $ref: '#/components/schemas/ToolTypes'
          default: function
        function:
          $ref: '#/components/schemas/FunctionCall'
        index:
          type: integer
          title: Index
          default: 0
      additionalProperties: false
      type: object
      required:
        - function
      title: ToolCall
    ToolChoice:
      properties:
        type:
          $ref: '#/components/schemas/ToolTypes'
          default: function
        function:
          $ref: '#/components/schemas/FunctionName'
      additionalProperties: false
      type: object
      required:
        - function
      title: ToolChoice
      description: ToolChoice is either a ToolChoiceEnum or a ToolChoice
    ToolChoiceEnum:
      type: string
      enum:
        - auto
        - none
        - any
        - required
      title: ToolChoiceEnum
    ToolMessage:
      properties:
        content:
          title: Content
          anyOf:
            - type: string
            - type: 'null'
            - items:
                $ref: '#/components/schemas/ContentChunk'
              type: array
        tool_call_id:
          anyOf:
            - type: string
            - type: 'null'
          title: Tool Call Id
        name:
          anyOf:
            - type: string
            - type: 'null'
          title: Name
        role:
          type: string
          default: tool
          enum:
            - tool
      additionalProperties: false
      type: object
      required:
        - content
      title: ToolMessage
    ToolTypes:
      type: string
      enum:
        - function
      const: function
      title: ToolTypes
    UserMessage:
      properties:
        content:
          title: Content
          anyOf:
            - type: string
            - type: 'null'
            - items:
                $ref: '#/components/schemas/ContentChunk'
              type: array
        role:
          type: string
          default: user
          enum:
            - user
      additionalProperties: false
      type: object
      required:
        - content
      title: UserMessage
    AgentsCompletionRequest:
      properties:
        max_tokens:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Max Tokens
          description: >-
            The maximum number of tokens to generate in the completion. The
            token count of your prompt plus `max_tokens` cannot exceed the
            model's context length.
        stream:
          type: boolean
          title: Stream
          default: false
          description: >-
            Whether to stream back partial progress. If set, tokens will be sent
            as data-only server-side events as they become available, with the
            stream terminated by a data: [DONE] message. Otherwise, the server
            will hold the request open until the timeout or until completion,
            with the response containing the full result as JSON.
        stop:
          anyOf:
            - type: string
            - items:
                type: string
              type: array
          title: Stop
          description: >-
            Stop generation if this token is detected. Or if one of these tokens
            is detected when providing an array
        random_seed:
          anyOf:
            - type: integer
              minimum: 0
            - type: 'null'
          title: Random Seed
          description: >-
            The seed to use for random sampling. If set, different calls will
            generate deterministic results.
        messages:
          items:
            oneOf:
              - $ref: '#/components/schemas/SystemMessage'
              - $ref: '#/components/schemas/UserMessage'
              - $ref: '#/components/schemas/AssistantMessage'
              - $ref: '#/components/schemas/ToolMessage'
            discriminator:
              propertyName: role
              mapping:
                assistant: '#/components/schemas/AssistantMessage'
                system: '#/components/schemas/SystemMessage'
                tool: '#/components/schemas/ToolMessage'
                user: '#/components/schemas/UserMessage'
          type: array
          title: Messages
          description: >-
            The prompt(s) to generate completions for, encoded as a list of dict
            with role and content.
          examples:
            - - role: user
                content: Who is the best French painter? Answer in one short sentence.
        response_format:
          $ref: '#/components/schemas/ResponseFormat'
        tools:
          anyOf:
            - items:
                $ref: '#/components/schemas/Tool'
              type: array
            - type: 'null'
          title: Tools
        tool_choice:
          anyOf:
            - $ref: '#/components/schemas/ToolChoice'
            - $ref: '#/components/schemas/ToolChoiceEnum'
          title: Tool Choice
          default: auto
        presence_penalty:
          type: number
          maximum: 2
          minimum: -2
          title: Presence Penalty
          default: 0
          description: >-
            presence_penalty determines how much the model penalizes the
            repetition of words or phrases. A higher presence penalty encourages
            the model to use a wider variety of words and phrases, making the
            output more diverse and creative.
        frequency_penalty:
          type: number
          maximum: 2
          minimum: -2
          title: Frequency Penalty
          default: 0
          description: >-
            frequency_penalty penalizes the repetition of words based on their
            frequency in the generated text. A higher frequency penalty
            discourages the model from repeating words that have already
            appeared frequently in the output, promoting diversity and reducing
            repetition.
        'n':
          anyOf:
            - type: integer
              minimum: 1
            - type: 'null'
          title: 'N'
          description: >-
            Number of completions to return for each request, input tokens are
            only billed once.
        prediction:
          $ref: '#/components/schemas/Prediction'
          default:
            type: content
            content: ''
          description: >-
            Enable users to specify expected results, optimizing response times
            by leveraging known or predictable content. This approach is
            especially effective for updating text documents or code files with
            minimal changes, reducing latency while maintaining high-quality
            results.
        agent_id:
          type: string
          description: The ID of the agent to use for this completion.
      additionalProperties: false
      type: object
      required:
        - messages
        - agent_id
      title: AgentsCompletionRequest
    ContentChunk:
      oneOf:
        - $ref: '#/components/schemas/TextChunk'
        - $ref: '#/components/schemas/ImageURLChunk'
        - $ref: '#/components/schemas/ReferenceChunk'
      discriminator:
        propertyName: type
        mapping:
          image_url: '#/components/schemas/ImageURLChunk'
          text: '#/components/schemas/TextChunk'
          reference: '#/components/schemas/ReferenceChunk'
      title: ContentChunk
    EmbeddingRequest:
      properties:
        input:
          anyOf:
            - type: string
            - items:
                type: string
              type: array
          title: Input
          description: Text to embed.
          example:
            - Embed this sentence.
            - As well as this one.
        model:
          type: string
          title: Model
          description: ID of the model to use.
          default: mistral-embed
        encoding_format:
          anyOf:
            - type: string
            - type: 'null'
          title: Encoding Format
          description: The format to return the embeddings in.
          default: float
      additionalProperties: false
      type: object
      required:
        - input
        - model
      title: EmbeddingRequest
    ChatClassificationRequest:
      properties:
        input:
          anyOf:
            - items:
                oneOf:
                  - $ref: '#/components/schemas/SystemMessage'
                  - $ref: '#/components/schemas/UserMessage'
                  - $ref: '#/components/schemas/AssistantMessage'
                  - $ref: '#/components/schemas/ToolMessage'
                discriminator:
                  propertyName: role
                  mapping:
                    assistant: '#/components/schemas/AssistantMessage'
                    system: '#/components/schemas/SystemMessage'
                    tool: '#/components/schemas/ToolMessage'
                    user: '#/components/schemas/UserMessage'
              type: array
            - items:
                items:
                  oneOf:
                    - $ref: '#/components/schemas/SystemMessage'
                    - $ref: '#/components/schemas/UserMessage'
                    - $ref: '#/components/schemas/AssistantMessage'
                    - $ref: '#/components/schemas/ToolMessage'
                  discriminator:
                    propertyName: role
                    mapping:
                      assistant: '#/components/schemas/AssistantMessage'
                      system: '#/components/schemas/SystemMessage'
                      tool: '#/components/schemas/ToolMessage'
                      user: '#/components/schemas/UserMessage'
                type: array
              type: array
          title: Input
          description: Chat to classify
        model:
          anyOf:
            - type: string
            - type: 'null'
          title: Model
      additionalProperties: false
      type: object
      required:
        - input
        - model
      title: ChatClassificationRequest
    ClassificationRequest:
      properties:
        input:
          anyOf:
            - type: string
            - items:
                type: string
              type: array
          title: Input
          description: Text to classify.
        model:
          anyOf:
            - type: string
            - type: 'null'
          title: Model
      additionalProperties: false
      type: object
      required:
        - input
      title: ClassificationRequest
    UsageInfo:
      title: UsageInfo
      type: object
      properties:
        prompt_tokens:
          type: integer
          example: 16
        completion_tokens:
          type: integer
          example: 34
        total_tokens:
          type: integer
          example: 50
      required:
        - prompt_tokens
        - completion_tokens
        - total_tokens
    ResponseBase:
      type: object
      title: ResponseBase
      properties:
        id:
          type: string
          example: cmpl-e5cc70bb28c444948073e77776eb30ef
        object:
          type: string
          example: chat.completion
        model:
          type: string
          example: mistral-small-latest
        usage:
          $ref: '#/components/schemas/UsageInfo'
    ChatCompletionChoice:
      title: ChatCompletionChoice
      type: object
      required:
        - index
        - finish_reason
        - message
      properties:
        index:
          type: integer
          example: 0
        message:
          $ref: '#/components/schemas/AssistantMessage'
        finish_reason:
          type: string
          enum:
            - stop
            - length
            - model_length
            - error
            - tool_calls
          example: stop
    DeltaMessage:
      title: DeltaMessage
      type: object
      properties:
        role:
          anyOf:
            - type: string
            - type: 'null'
        content:
          anyOf:
            - type: string
            - type: 'null'
            - items:
                $ref: '#/components/schemas/ContentChunk'
              type: array
        tool_calls:
          anyOf:
            - type: 'null'
            - type: array
              items:
                $ref: '#/components/schemas/ToolCall'
    ChatCompletionResponseBase:
      allOf:
        - $ref: '#/components/schemas/ResponseBase'
        - type: object
          title: ChatCompletionResponseBase
          properties:
            created:
              type: integer
              example: 1702256327
    ChatCompletionResponse:
      allOf:
        - $ref: '#/components/schemas/ChatCompletionResponseBase'
        - type: object
          title: ChatCompletionResponse
          properties:
            choices:
              type: array
              items:
                $ref: '#/components/schemas/ChatCompletionChoice'
          required:
            - id
            - object
            - data
            - model
            - usage
    FIMCompletionResponse:
      allOf:
        - $ref: '#/components/schemas/ChatCompletionResponse'
        - type: object
          properties:
            model:
              type: string
              example: codestral-latest
    EmbeddingResponseData:
      title: EmbeddingResponseData
      type: object
      properties:
        object:
          type: string
          example: embedding
        embedding:
          type: array
          items:
            type: number
          example:
            - 0.1
            - 0.2
            - 0.3
        index:
          type: integer
          example: 0
      examples:
        - object: embedding
          embedding:
            - 0.1
            - 0.2
            - 0.3
          index: 0
        - object: embedding
          embedding:
            - 0.4
            - 0.5
            - 0.6
          index: 1
    EmbeddingResponse:
      allOf:
        - $ref: '#/components/schemas/ResponseBase'
        - type: object
          properties:
            data:
              type: array
              items:
                - $ref: '#/components/schemas/EmbeddingResponseData'
          required:
            - id
            - object
            - data
            - model
            - usage
    ClassificationResponse:
      type: object
      title: ClassificationResponse
      properties:
        id:
          type: string
          example: mod-e5cc70bb28c444948073e77776eb30ef
        model:
          type: string
        results:
          type: array
          items:
            - $ref: '#/components/schemas/ClassificationObject'
    ClassificationObject:
      type: object
      title: ClassificationObject
      properties:
        categories:
          description: Classifier result thresholded
          type: object
          additionalProperties:
            type: boolean
        category_scores:
          description: Classifier result
          type: object
          additionalProperties:
            type: number
    CompletionEvent:
      title: CompletionEvent
      type: object
      required:
        - data
      properties:
        data:
          $ref: '#/components/schemas/CompletionChunk'
    CompletionChunk:
      title: CompletionChunk
      type: object
      required:
        - id
        - model
        - choices
      properties:
        id:
          type: string
        object:
          type: string
        created:
          type: integer
        model:
          type: string
        usage:
          $ref: '#/components/schemas/UsageInfo'
        choices:
          type: array
          items:
            $ref: '#/components/schemas/CompletionResponseStreamChoice'
    CompletionResponseStreamChoice:
      title: CompletionResponseStreamChoice
      type: object
      required:
        - index
        - delta
        - finish_reason
      properties:
        index:
          type: integer
        delta:
          $ref: '#/components/schemas/DeltaMessage'
        finish_reason:
          type:
            - string
            - 'null'
          enum:
            - stop
            - length
            - error
            - tool_calls
            - null
  securitySchemes:
    ApiKey:
      type: http
      scheme: bearer
tags:
  - name: chat
    x-displayName: Chat
    description: Chat Completion API.
  - name: fim
    x-displayName: FIM
    description: Fill-in-the-middle API.
  - name: agents
    x-displayName: Agents
    description: Agents API.
  - name: embeddings
    x-displayName: Embeddings
    description: Embeddings API.
  - name: classifiers
    x-displayName: Classifiers
    description: Classifiers API.
  - name: files
    x-displayName: Files
    description: Files API
  - name: fine-tuning
    x-displayName: Fine Tuning
    description: Fine-tuning API
  - name: models
    x-displayName: Models
    description: Model Management API
  - name: batch
    x-displayName: Batch
    description: Batch API
security:
  - ApiKey: []
servers:
  - url: https://api.mistral.ai
    description: Production server
