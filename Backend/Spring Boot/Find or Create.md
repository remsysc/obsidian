
```
private Set<Tag> resolveOrCreateTags(Set<String> tagNames) 
{ return tagNames.stream()
 .map(name -> name.toLowerCase().trim()) 
 .map(name -> tagRepository.findByName(name) 
 .orElseGet(() -> 
 tagRepository.save( Tag.builder().name(name).build() ))) .collect(Collectors.toSet()); } ``` --- ## Key Concepts Learned 
### 1. Find Or Create Pattern  Tag exists? → return existing tag Tag missing? → create it, return new tag One query, no duplication
```

if a business model needs to created when they needed

[[Understand the Business Requirements & Logic]]]