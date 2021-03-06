## REST API HANDLERS SPIRNG
```java
@PostMapping
	public void createTopic(@Valid @RequestBody Topic topic) {
		LOGEER.info("POST /topics ", topic.getId());
		topicservice.addTopic(topic); // first verify if it doen't exist then only create
		//return new ResponseEntity<Topic>(topic, HttpStatus.OK);
	}
  
```java
@GetMapping("/{id}")
	public Topic getTopicbyId(@PathVariable("id") String id) {
		LOGEER.info("GET /topics/{} "+ id);
		return topicservice.getTopic(id).orElseThrow(() -> new ResourceNotFoundException("Topic does not exist: " + id));
	}
  
  @GetMapping
	public List<Topic> getAllTopic() {
		LOGEER.info("GET /topics ");
		return topicservice.getAllTopics();
	}
  
  	@PutMapping("/{id}")
	public ResponseEntity<?> updateTopic(@PathVariable String id, @Valid @RequestBody Topic newTopic) {
		LOGEER.info("PUT /topics/{} "+ id);
		Topic topic = topicservice.getTopic(id).orElseThrow(() -> new ResourceNotFoundException("Topic does not exist: " + id));
		topic.setDescription(newTopic.getDescription());
		topic.setName(newTopic.getName());
		topicservice.updateTopic(id, topic);
		return new ResponseEntity("Updated succesfully " , HttpStatus.OK );
	}
  
  @DeleteMapping("/{id}")
	public ResponseEntity<?> deleteTopic(@PathVariable("id") String id) {
		LOGEER.info("DELETE /topics/{} "+ id);
		Topic topic = topicservice.getTopic(id).orElseThrow(() -> new ResourceNotFoundException("Topic does not exist: " + id));
		topicservice.deleteTopic(id);
		return ResponseEntity.ok("Topic with id "+ id +" deleted successfully.");
	}

@RequestMapping(value = "/authenticate", method = RequestMethod.POST)
	public ResponseEntity<?> createAuthenticationToken(@RequestBody AuthenticationRequest authenticationRequest)
			throws Exception {
		try {
			authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(
					authenticationRequest.getUsername(), authenticationRequest.getPassword()));
		} catch (BadCredentialsException e) {
			throw new Exception("Incorrect username or password", e);
		}
		final UserDetails userDetails = userDetailsService.loadUserByUsername(authenticationRequest.getUsername());
		final String jwt = jwtTokenUtil.generateToken(userDetails);
		return ResponseEntity.ok(new AuthenticationResponse(jwt));
	}
  
  
Docker Commands :
image for java 11
  FROM openjdk:11
  VOLUME /tmp
  ADD target/prodready-0.0.1-SNAPSHOT.jar app.jar
  ENTRYPOINT ["java", "-jar", "/app.jar"]
  
  Docker ???version
  Docker build -t nameofimageyouwant:v1 .
  List image s: docker image ls
  
  Run container : docker run -p8080:8080 imageName:tag
  
  Check running/active containers : docker ps 
  Check all containers  active/stopped/inactive : docker ps -a
  Stop container. : docker container stop {conatainerId}
  Start container : docker container start id
  
Stop and remove all containers:
  docker stop $(docker ps -a -q)
  docker rm $(docker ps -a -q)


SpringBoot Test class Example:
import static org.junit.Assert.assertEquals;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import org.junit.jupiter.api.Test;
//import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.firstsample.domain.Topic;
import com.example.firstsample.repo.TopicRepository;

@SpringBootTest
public class TopicServiceTest {
	@Autowired 
	private TopicService topicService;
	
	@MockBean
	private TopicRepository topicRepository;
	
	@Test
	public void getTopicsTest(){
		when(topicRepository.findAll()).thenReturn(Stream.of(new Topic("1","java","test1"))
				.collect(Collectors.toList()));
		assertEquals(1, topicService.getAllTopics().size());
		
	}
	@Test
	public void addTopicTest() {
		Topic topic = new Topic("12", "java", "test2");
		topicService.addTopic(topic);
		verify(topicRepository , times(1)).save(topic);
	}
	
	@Test
	public void getTopicByIdTest() {
		String topicId = "12";
		when(topicRepository.findById(topicId))
		.thenReturn(Optional.of(new Topic("12","java","test1")));
		assertEquals("12", topicService.getTopic(topicId).get().getId());
		assertEquals("java", topicService.getTopic(topicId).get().getName());
		assertEquals("test1", topicService.getTopic(topicId).get().getDescription());
	}
	
	@Test
	public void deleteTopicTest() {
		Topic topic = new Topic("12", "java", "test2");
		topicService.deleteTopic(topic.getId());
		verify(topicRepository , times(1)).deleteById(topic.getId());
	}
}

