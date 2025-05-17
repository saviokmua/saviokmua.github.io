# 

# Conditional Routing in Ruby on Rails Using Custom Constraints

In complex Ruby on Rails applications, it's common to route requests based on dynamic conditions, such as the value of a query parameter. One powerful way to handle such cases is to use **custom routing constraints**. This article demonstrates how to route the same endpoint to different controllers based on the `subject_type` parameter.

## Problem

Suppose we want to handle updates to details of various animal types—dogs, cats, and mice—through a single endpoint like:

```
PUT /details?subject_type=Dog
```

Instead of writing separate endpoints or bloating a single controller with conditional logic, we can delegate requests to separate controllers using Rails' routing constraints.

## Solution: Use a Custom Constraint

We define a custom constraint class that inspects the incoming request:

```ruby
class SubjectTypeConstraints
  attr_reader :subject_type

  def initialize(options)
    @subject_type = options[:subject_type]
  end

  def matches?(req)
    req.params['subject_type'] == subject_type
  end
end
```

This class will be used to match a specific `subject_type` query parameter in the request.

## Routing Setup

Using the `constraints` option, we define three routes under the same path and HTTP method, but each constrained to a specific `subject_type` value:

```ruby
resources :details, only: :none do
  put :update, on: :collection,
      controller:  '/api/dogs/details',
      constraints: SubjectTypeConstraints.new(subject_type: 'Dog')

  put :update, on: :collection,
      controller:  '/api/cats/details',
      constraints: SubjectTypeConstraints.new(subject_type: 'Cat')

  put :update, on: :collection,
      controller:  '/api/mice/details',
      constraints: SubjectTypeConstraints.new(subject_type: 'Mouse')
end
```

This sets up three routes that all respond to `PUT /details`, but depending on the `subject_type` parameter (`Dog`, `Cat`, or `Mouse`), the request will be routed to the appropriate controller.

## Benefits

- **Clean Separation of Concerns**: Each animal type has its own controller and logic.
- **Scalability**: Easily extendable to support more types without complicating existing code.
- **RESTful Compliance**: Maintains a single endpoint path while offering internal routing flexibility.

## Example Requests

| Request                            | Routed Controller               |
|------------------------------------|---------------------------------|
| `PUT /details?subject_type=Dog`    | `Api::Dogs::DetailsController`  |
| `PUT /details?subject_type=Cat`    | `Api::Cats::DetailsController`  |
| `PUT /details?subject_type=Mouse`  | `Api::Mice::DetailsController`  |

## Considerations

- If no `subject_type` is provided or it doesn't match any of the constraints, Rails will return a 404.
- Ensure controllers follow a consistent interface (e.g., same action names) to reduce confusion and duplication.
- You can further enhance the constraint logic to support headers, request body inspection, or authentication.

## Conclusion

Custom route constraints in Rails offer a powerful and elegant way to handle conditional routing. By leveraging query parameters and clean controller separation, you can keep your code modular, readable, and maintainable.

