pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.test</groupId>
	<artifactId>mockito-learning</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.mockito</groupId>
			<artifactId>mockito-all</artifactId>
			<version>1.10.19</version>
			<scope>test</scope>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.powermock/powermock-api-mockito -->
		<dependency>
			<groupId>org.powermock</groupId>
			<artifactId>powermock-api-mockito</artifactId>
			<version>1.6.5</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.powermock</groupId>
			<artifactId>powermock-module-junit4</artifactId>
			<version>1.6.5</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.hamcrest</groupId>
			<artifactId>hamcrest-library</artifactId>
			<version>1.3</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
```
readme.md
```
```
src\test\java\com\mockito\examples\BasicMockitoTest.java
```
package com.mockito.examples;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNull;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.junit.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Mockito;

public class BasicMockitoTest {

	// External Service - Lets say this comes from wunderlist
	interface TodoService {
		public List<String> retrieveTodos(String user);

		void deleteTodo(String todo);
	}

	class TodoBusinessImpl {
		private TodoService todoService;

		TodoBusinessImpl(TodoService todoService) {
			this.todoService = todoService;
		}

		public List<String> retrieveTodosRelatedToSpring(String user) {
			List<String> filteredTodos = new ArrayList<String>();
			List<String> allTodos = todoService.retrieveTodos(user);
			for (String todo : allTodos) {
				if (todo.contains("Spring")) {
					filteredTodos.add(todo);
				}
			}
			return filteredTodos;
		}

		public void deleteTodosNotRelatedToSpring(String user) {
			List<String> allTodos = todoService.retrieveTodos(user);
			for (String todo : allTodos) {
				if (!todo.contains("Spring")) {
					todoService.deleteTodo(todo);
				}
			}
		}

	}

	class TodoServiceStub implements TodoService {
		public List<String> retrieveTodos(String user) {
			return Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		}

		public void deleteTodo(String todo) {
			// TODO Auto-generated method stub
		}
	}

	@Test
	public void usingAStub() {
		TodoService todoService = new TodoServiceStub();
		TodoBusinessImpl todoBusinessImpl = new TodoBusinessImpl(todoService);
		List<String> todos = todoBusinessImpl.retrieveTodosRelatedToSpring("Ranga");
		assertEquals(2, todos.size());
	}

	@Test
	public void usingAMock() {
		TodoService todoService = mock(TodoService.class);
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		Mockito.when(todoService.retrieveTodos("Ranga")).thenReturn(allTodos);
		TodoBusinessImpl todoBusinessImpl = new TodoBusinessImpl(todoService);
		List<String> todos = todoBusinessImpl.retrieveTodosRelatedToSpring("Ranga");
		assertEquals(2, todos.size());
	}

	@Test
	public void letsMockListSize() {
		List list = mock(List.class);
		Mockito.when(list.size()).thenReturn(10);
		assertEquals(10, list.size());
	}

	@Test
	public void letsMockListSizeWithMultipleReturnValues() {
		List list = mock(List.class);
		Mockito.when(list.size()).thenReturn(10).thenReturn(20);
		assertEquals(10, list.size()); // First Call
		assertEquals(20, list.size()); // Second Call
	}

	@Test
	public void letsMockListGet() {
		List<String> list = mock(List.class);
		Mockito.when(list.get(0)).thenReturn("in28Minutes");
		assertEquals("in28Minutes", list.get(0));
		assertNull(list.get(1));
	}

	@Test
	public void letsMockListGetWithAny() {
		List<String> list = mock(List.class);
		Mockito.when(list.get(Mockito.anyInt())).thenReturn("in28Minutes");
		// If you are using argument matchers, all arguments
		// have to be provided by matchers.
		assertEquals("in28Minutes", list.get(0));
		assertEquals("in28Minutes", list.get(1));
	}

	@Test
	public void bddAliases_UsingGivenWillReturn() {
		List<String> list = mock(List.class);
		given(list.get(Mockito.anyInt())).willReturn("in28Minutes");
		// If you are using argument matchers, all arguments
		// have to be provided by matchers.
		assertEquals("in28Minutes", list.get(0));
		assertEquals("in28Minutes", list.get(1));
	}

	@Test(expected = RuntimeException.class)
	public void letsMockListGetToThrowException() {
		List<String> list = mock(List.class);
		Mockito.when(list.get(Mockito.anyInt())).thenThrow(new RuntimeException("Something went wrong"));
		list.get(0);
	}

	@Test
	public void letsTestDeleteNow() {
		TodoService todoService = mock(TodoService.class);
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		Mockito.when(todoService.retrieveTodos("Ranga")).thenReturn(allTodos);

		TodoBusinessImpl todoBusinessImpl = new TodoBusinessImpl(todoService);

		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");

		Mockito.verify(todoService).deleteTodo("Learn to Dance");
		Mockito.verify(todoService, Mockito.never()).deleteTodo("Learn Spring MVC");
		Mockito.verify(todoService, Mockito.never()).deleteTodo("Learn Spring");

		Mockito.verify(todoService, Mockito.times(1)).deleteTodo("Learn to Dance");
		// atLeastOnce, atLeast

	}

	@Test
	public void captureArgument() {
		ArgumentCaptor<String> argumentCaptor = ArgumentCaptor.forClass(String.class);

		TodoService todoService = mock(TodoService.class);

		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		Mockito.when(todoService.retrieveTodos("Ranga")).thenReturn(allTodos);

		TodoBusinessImpl todoBusinessImpl = new TodoBusinessImpl(todoService);
		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");
		Mockito.verify(todoService).deleteTodo(argumentCaptor.capture());

		assertEquals("Learn to Dance", argumentCaptor.getValue());
	}
}
```
src\test\java\com\mockito\examples\HamcrestMatcherTest.java
```
package com.mockito.examples;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.arrayContainingInAnyOrder;
import static org.hamcrest.Matchers.arrayWithSize;
import static org.hamcrest.Matchers.greaterThan;
import static org.hamcrest.Matchers.hasItems;
import static org.hamcrest.Matchers.isEmptyOrNullString;
import static org.hamcrest.Matchers.isEmptyString;
import static org.hamcrest.Matchers.lessThan;
import static org.hamcrest.collection.IsCollectionWithSize.hasSize;
import static org.hamcrest.core.Every.everyItem;

import java.util.Arrays;
import java.util.List;

import org.junit.Test;

public class HamcrestMatcherTest {

	@Test
	public void basicHamcrestMatchers() {
		List<Integer> scores = Arrays.asList(99, 100, 101, 105);
		assertThat(scores, hasSize(4));
		assertThat(scores, hasItems(100, 101));
		assertThat(scores, everyItem(greaterThan(90)));
		assertThat(scores, everyItem(lessThan(200)));

		// String
		assertThat("", isEmptyString());
		assertThat(null, isEmptyOrNullString());

		// Array
		Integer[] marks = { 1, 2, 3 };

		assertThat(marks, arrayWithSize(3));
		assertThat(marks, arrayContainingInAnyOrder(2, 3, 1));

	}
}
```
src\test\java\com\mockito\examples\InjectMocksTest.java
```
package com.mockito.examples;

import static org.junit.Assert.assertEquals;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;

// External Service - Lets say this comes from wunderlist
interface TodoService {
	public List<String> retrieveTodos(String user);

	void deleteTodo(String todo);
}

class TodoBusinessImpl {
	private TodoService todoService;

	TodoBusinessImpl(TodoService todoService) {
		this.todoService = todoService;
	}

	public List<String> retrieveTodosRelatedToSpring(String user) {
		List<String> filteredTodos = new ArrayList<String>();
		List<String> allTodos = todoService.retrieveTodos(user);
		for (String todo : allTodos) {
			if (todo.contains("Spring")) {
				filteredTodos.add(todo);
			}
		}
		return filteredTodos;
	}

	public void deleteTodosNotRelatedToSpring(String user) {
		List<String> allTodos = todoService.retrieveTodos(user);
		for (String todo : allTodos) {
			if (!todo.contains("Spring")) {
				todoService.deleteTodo(todo);
			}
		}
	}

}

@RunWith(MockitoJUnitRunner.class)
public class InjectMocksTest {

	@Mock
	TodoService todoServiceMock;

	@InjectMocks
	TodoBusinessImpl todoBusinessImpl;

	@Captor
	ArgumentCaptor<String> stringArgumentCaptor;

	@Test
	public void usingAMock() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		List<String> todos = todoBusinessImpl.retrieveTodosRelatedToSpring("Ranga");
		assertEquals(2, todos.size());
	}

	@Test
	public void letsMockListSize() {
		List<String> list = mock(List.class);
		when(list.size()).thenReturn(10);
		assertEquals(10, list.size());
	}

	@Test
	public void letsMockListSizeWithMultipleReturnValues() {
		List<String> list = mock(List.class);
		when(list.size()).thenReturn(10).thenReturn(20);
		assertEquals(10, list.size()); // First Call
		assertEquals(20, list.size()); // Second Call
	}

	@Test
	public void letsTestDeleteNow() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");

		verify(todoServiceMock).deleteTodo("Learn to Dance");
		verify(todoServiceMock, never()).deleteTodo("Learn Spring MVC");
		verify(todoServiceMock, never()).deleteTodo("Learn Spring");

		verify(todoServiceMock, times(1)).deleteTodo("Learn to Dance");
		// atLeastOnce, atLeast

	}

	@Test
	public void captureArgument() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");
		verify(todoServiceMock).deleteTodo(stringArgumentCaptor.capture());

		assertEquals("Learn to Dance", stringArgumentCaptor.getValue());
	}
	// Spy
	// Mocking final and static methods with PowerMock
}
```
src\test\java\com\mockito\examples\MockitoRulesTest.java
```
package com.mockito.examples;

import static org.junit.Assert.assertEquals;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import java.util.Arrays;
import java.util.List;

import org.junit.Rule;
import org.junit.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnit;
import org.mockito.junit.MockitoRule;

public class MockitoRulesTest {

	@Rule
	public MockitoRule mockitoRule = MockitoJUnit.rule();

	@Mock
	TodoService todoServiceMock;

	@InjectMocks
	TodoBusinessImpl todoBusinessImpl;

	@Captor
	ArgumentCaptor<String> stringArgumentCaptor;

	@Test
	public void usingAMock() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		List<String> todos = todoBusinessImpl.retrieveTodosRelatedToSpring("Ranga");
		assertEquals(2, todos.size());
	}

	@Test
	public void letsMockListSize() {
		List<String> list = mock(List.class);
		when(list.size()).thenReturn(10);
		assertEquals(10, list.size());
	}

	@Test
	public void letsMockListSizeWithMultipleReturnValues() {
		List<String> list = mock(List.class);
		when(list.size()).thenReturn(10).thenReturn(20);
		assertEquals(10, list.size()); // First Call
		assertEquals(20, list.size()); // Second Call
	}

	@Test
	public void letsTestDeleteNow() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");

		verify(todoServiceMock).deleteTodo("Learn to Dance");
		verify(todoServiceMock, never()).deleteTodo("Learn Spring MVC");
		verify(todoServiceMock, never()).deleteTodo("Learn Spring");

		verify(todoServiceMock, times(1)).deleteTodo("Learn to Dance");
		// atLeastOnce, atLeast

	}

	@Test
	public void captureArgument() {
		List<String> allTodos = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn to Dance");
		when(todoServiceMock.retrieveTodos("Ranga")).thenReturn(allTodos);

		todoBusinessImpl.deleteTodosNotRelatedToSpring("Ranga");
		verify(todoServiceMock).deleteTodo(stringArgumentCaptor.capture());

		assertEquals("Learn to Dance", stringArgumentCaptor.getValue());
	}
}
```
src\test\java\com\mockito\examples\PowerMockitoTest.java
```
package com.mockito.examples;

import static org.junit.Assert.assertEquals;
import static org.mockito.Matchers.anyLong;
import static org.mockito.Mockito.when;

import java.util.Arrays;
import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import org.powermock.reflect.Whitebox;

class StaticClass {
	static int staticMethod(long value) {
		// Some complex logic is done here...
		throw new RuntimeException("Something I need is not available");
	}
}

interface Dependency {
	List<Integer> retrieveAllStats();
}

class SystemUnderTest {
	Dependency dependency;

	public int methodUnderTest() {
		List<Integer> stats = dependency.retrieveAllStats();
		long sum = 0;
		for (int stat : stats)
			sum += stat;
		return StaticClass.staticMethod(sum);
	}

	private int privateMethodUnderTest() {
		List<Integer> stats = dependency.retrieveAllStats();
		long sum = 0;
		for (int stat : stats)
			sum += stat;
		return StaticClass.staticMethod(sum);
	}

}

@RunWith(PowerMockRunner.class)
@PrepareForTest(StaticClass.class)
public class PowerMockitoTest {

	@Mock
	Dependency dependencyMock;

	@InjectMocks
	SystemUnderTest systemUnderTest;

	@Test
	public void powerMockito_MockingAStaticMethodCall() {

		PowerMockito.mockStatic(StaticClass.class);

		when(dependencyMock.retrieveAllStats()).thenReturn(Arrays.asList(1, 2, 3));
		when(StaticClass.staticMethod(anyLong())).thenReturn(150);

		assertEquals(150, systemUnderTest.methodUnderTest());

		PowerMockito.verifyStatic();
		StaticClass.staticMethod(1 + 2 + 3);

		// verify exact number of calls
		// PowerMockito.verifyStatic(Mockito.times(2)):

	}

	@Test
	public void powerMockito_CallingAPrivateMethod() throws Exception {
		PowerMockito.mockStatic(StaticClass.class);
		when(dependencyMock.retrieveAllStats()).thenReturn(Arrays.asList(1, 2, 3));
		when(StaticClass.staticMethod(Mockito.anyLong())).thenReturn(150);
		int value = (Integer) Whitebox.invokeMethod(systemUnderTest, "privateMethodUnderTest");
		assertEquals(150, value);
	}

}
```
src\test\java\com\mockito\examples\SpyTest.java
```
package com.mockito.examples;

import static org.junit.Assert.assertEquals;
import static org.mockito.Mockito.spy;
import static org.mockito.Mockito.stub;
import static org.mockito.Mockito.verify;

import java.util.ArrayList;
import java.util.List;

import org.junit.Test;

public class SpyTest {

	@Test
	public void creatingASpyOnArrayList() {
		List<String> listSpy = spy(ArrayList.class);
		listSpy.add("Ranga");
		listSpy.add("in28Minutes");

		verify(listSpy).add("Ranga");
		verify(listSpy).add("in28Minutes");

		assertEquals(2, listSpy.size());
		assertEquals("Ranga", listSpy.get(0));
	}

	@Test
	public void creatingASpyOnArrayList_overridingSpecificMethods() {
		List<String> listSpy = spy(ArrayList.class);
		listSpy.add("Ranga");
		listSpy.add("in28Minutes");

		stub(listSpy.size()).toReturn(-1);

		assertEquals(-1, listSpy.size());
		assertEquals("Ranga", listSpy.get(0));

		// @Spy Annotation
	}

}
```
