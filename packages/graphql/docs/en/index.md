# Information

This extension provide integration of [Webonyx GraphQL](http://webonyx.github.io/graphql-php/) into Nette Framework.


## Installation

The simplest way to install Portiny/GraphQL is using  [Composer](http://getcomposer.org/):

```sh
$ composer require portiny/graphql
```

Enable the extension at your neon config file.

```yml
extensions:
    - Portiny\GraphQL\Adapter\Nette\DI\GraphQLExtension
```


## Usage

Now you can create new `Query` or `Mutation` class with interface `Portiny\GraphQL\Contract\Field\QueryFieldInterface` or `Portiny\GraphQL\Contract\Mutation\MutationFieldInterface`. Then register your class into DI container via your neon config file.

Example `Query` class may looks like this

```php
<?php

declare(strict_types = 1);

namespace App\GraphQL\Query;

use GraphQL\Type\Definition\Type;
use Portiny\GraphQL\Contract\Field\QueryFieldInterface;
use Portiny\GraphQL\GraphQL\Type\Scalar\EmailType;
use Portiny\GraphQL\GraphQL\Type\Types;


final class SimpleQueryField implements QueryFieldInterface
{

	/**
	 * {@inheritdoc}
	 */
	public function getName(): string
	{
		return 'simple';
	}


	/**
	 * {@inheritdoc}
	 */
	public function getType(): Type
	{
		return Type::string();
	}


	/**
	 * {@inheritdoc}
	 */
	public function getDescription(): string
	{
		return 'Some description for simple query.';
	}


	/**
	 * {@inheritdoc}
	 */
	public function getArgs(): array
	{
		return [
			'email' => ['type' => Types::get(EmailType::class)],
			'password' => ['type' => Type::string()]
		];
	}


	/**
    	 * {@inheritdoc}
    	 */
	public function resolve(array $root, array $args, $context = NULL)
	{
		// some logic for resolving query
		return 'resolved';
	}

}

```

then register it into DI container:

```yml
services:
    - App\GraphQL\Query\SimpleQueryField
```

### Presenter

To process GraphQL request we must use `Portiny\GraphQL\GraphQL\RequestProcessor` at presenter action.

```php
<?php

declare(strict_types = 1);

namespace App\Modules\GraphQLModule\FrontModule\Presenters;

use Nette\Application\UI\Presenter;
use Portiny\GraphQL\GraphQL\RequestProcessor;


class GraphQLPresenter extends Presenter
{

	/**
	 * @var RequestProcessor
	 */
	private $requestProcessor;


	public function __construct(RequestProcessor $requestProcessor) 
	{
		$this->requestProcessor = $requestProcessor;
	}


	public function actionDefault()
	{
		$this->sendJson(
			$this->requestProcessor->process()
		);
	}

}
```

## Additional parameters

### Root value

You can pass root values via `$requestProcessor->process(['someKey' => 'some value'])`. Root values will be passed into first argument at `resolve(array $root, array $args, $context = NULL)`.

### Context

You can create your own object with context and pass it via `$requestProcessor->process([], $myContextInstance)`; Context will be passed into third argument at `resolve(array $root, array $args, $context = NULL)`.

### Allowed queries

You can define allowed queries via `$requestProcessor->process([], NULL, [App\GraphQL\Query\SimpleQueryField::class])`. By default are allowed all registered queries.

### Allowed mutations

You can define allowed mutations via `$requestProcessor->process([], NULL, NULL, [App\GraphQL\Mutation\SimpleMutationField::class])`. By default are allowed all registered mutations.
