---
title: spring-boot-object-provider
date: 2021-09-03 23:28:23
tags:
categories:
thumbnail:
---

```java
public interface ObjectProvider<T> extends ObjectFactory<T>, Iterable<T> {

	T getObject(Object... args) throws BeansException;

	@Nullable
	T getIfAvailable() throws BeansException;

	default T getIfAvailable(Supplier<T> defaultSupplier) throws BeansException {
		T dependency = getIfAvailable();
		return (dependency != null ? dependency : defaultSupplier.get());
	}

	default void ifAvailable(Consumer<T> dependencyConsumer) throws BeansException {
		T dependency = getIfAvailable();
		if (dependency != null) {
			dependencyConsumer.accept(dependency);
		}
	}

	@Nullable
	T getIfUnique() throws BeansException;

	default T getIfUnique(Supplier<T> defaultSupplier) throws BeansException {
		T dependency = getIfUnique();
		return (dependency != null ? dependency : defaultSupplier.get());
	}

	default void ifUnique(Consumer<T> dependencyConsumer) throws BeansException {
		T dependency = getIfUnique();
		if (dependency != null) {
			dependencyConsumer.accept(dependency);
		}
	}

	@Override
	default Iterator<T> iterator() {
		return stream().iterator();
	}

	default Stream<T> stream() {
		throw new UnsupportedOperationException("Multi element access not supported");
	}

	default Stream<T> orderedStream() {
		throw new UnsupportedOperationException("Ordered element access not supported");
	}


```