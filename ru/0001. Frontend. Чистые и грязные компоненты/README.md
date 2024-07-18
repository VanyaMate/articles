# Frontend. Чистые и грязные компоненты

Здравствуйте :-)

Коротко о чем тут, чтобы вы могли понять нужно ли оно вам или нет.

Тут я описываю то к чему я пришел в разработке именно компонентов и какой подход использую. Материал может быть полезна скорее для новичков и не обязательно в React, потому что этот подход подойдет и для всего остального, но примеры на React.

Я уверен, что по этой теме было создано огромное количество материала, но вдруг в момент выхода моего — какой‑нибудь прошлый я прочтет это и о чем‑нибудь задумается. Если это поможет хотя бы одному — это было не зря.

Я не могу утверждать, что это правильное решение или что это в принципе хорошее решение для действительно больших проектов. Я не знаю. Но в тоже время, чисто логически, это выглядит как правильное решение и все последние проекты которые я делал с этим подходом — легко поддерживаются и разрабатываются.

А теперь к сути. В чем идея?

Я предлагаю разделить все компоненты на 4 типа.

## 1. Компоненты (Components).

Components - это чистые компоненты которые не зависят ни от чего из вне. Составляют бОльшую часть всех компонентов в приложении.

Особенности:

- Они никак не взаимодействуют ни с чем внешним, что не передается внутрь через пропсы.

- Если они используют внутри себя другие компоненты - то они тоже должны быть чистыми.

- Внутри разрешается использовать только чистые хуки (чистые функции).

- Желательно, когда возможно, любые сложные компоненты со своей логикой так же передавать через пропсы, но не обязательно.

Например:

```typescript jsx
// ...
	<ProductCard
		key={product.id}
		product={product}
		onClick={productCardClickHandler}
		header={<ImageSlider images={product.images}/>}
		footer={<AddToCartButton id={product.id} onClick={addToCartProduct}/>}
	/>
// ...
```

```typescript jsx
// Чистый компонент карточки ProductCard

export type ProductCardProps = {
	product: Product;
	onClick: (product: Product) => any;
	header?: React.ReactNode;
	footer?: React.ReactNode;
};

export const ProductCard: React.FC<ProductCardProps> = (props) => {
	const { product, onClick, header, footer } = props;
	const price: ProductPriceData = useProductPrice({
		price: product.price;
		discount: product.discount;
		discountType: product.discountType;
	});

	return (
		<div className={css.container} onClick={() => onClick(product)}>
			{ header }
			<div className={css.info}>
				<h3 className={css.title}>{ product.title }</h3>
				<p className={css.description}>{ product.description }</p>
				<ProductPrice price={price}/>
			</div>
			{ footer }
		</div>
	);
};
```
```typescript jsx
// Чистый компонент ProductPrice

export type ProductPriceProps = {
	price: ProductPriceData;
}

// Получает price и рендерит так же чистый компонент в зависимости от того
// есть ли скидка или нет
export const ProductPrice: React.FC<ProductPriceProps> = (props) => {
	const { price } = props;
	
	return price.discountPercent 
		? <ProductPriceWithDiscount price={price}/>
		: <ProductPriceWithoutDiscount price={price.price}/>
};
```
```typescript jsx
// Чистый хук useProductPrice для высчитывания цены товара и скидки

export type UseProductPriceProps = {
	price: number;
	discount: number;
	discountType: 'percent' | 'fixed';
}

export type ProductPriceData = {
	price: number;
	priceWithDiscount: number;
	discountPercent: number;
}

// По сути это просто чистая функция которая производит какие-то расчеты
export const useProductPrice = function (props: UseProductPriceProps): ProductPriceData {
	// .. Расчеты ..

	return useMemo(() => ({
		price, priceWithDiscount, discountPercent
	}), [price, priceWithDiscount, discountPercent]);
};
```

Сделав эти компоненты чистыми - мы получаем очень большую гибкость и легкость поддержки.

Мы можем спокойно сделать, что при клике на карточку - мы перейдем на другую страницу или откроем превью товара или что угодно другое, не важно.

Мы можем легко поменять слайдер карточки на новый или заменить старую кнопку добавления в корзину на новую. Это становится очень просто.

И никакие из этих, казалось бы, больших изменений, не затрагивают код в карточке. Мы можем спокойно, независимо, разрабатывать любой сложности компоненты и добавлять их внутрь ProductCard через header/footer (в нашем примере). И это могут быть даже грязные компоненты, если необходимо. В случае, если мы передаем их через пропсы - это допустимо, ведь компонент никак от них не зависит.

Так же мы можем их легко тестировать.

Так же меньше саморендеров.

Вы можете сказать: “Так это же очевидно, Вань”. Ну а я только недавно до этого дошел.

Давайте приведу условный пример кода, который я писал до этого и вы сразу поймете о чем я.
```typescript jsx
export type ProductCardProps = {
	product: Product;
};

export const ProductCard: React.FC<ProductCardProps> = (props) => {
	const { product } = props;
	{ /* зависимость */ }
	const navigate = useNavigate();
	const price: ProductPriceData = useProductPrice({
		price: product.price;
		discount: product.discount;
		discountType: product.discountType;
	});

	return (
		{ /* зависимость. теперь тут всегда ссылка и всегда на /product/*/ }
		<div className={css.container} onClick={navigate('/product/' + product.id)}>
			<ImageSlider images={product.images}/>
			<div className={css.info}>
				<h3 className={css.title}>{ product.title }</h3>
				<p className={css.description}>{ product.description }</p>
				<ProductPrice price={price}/>
			</div>
			{ /* зависимость потому что это грязный компонент с зависимостями */ }
			<AddToCartButton id={product.id}/>
		</div>
	);
};
```
```typescript jsx
export type AddToCartButtonProps = {
	id: string;
}

export const AddToCartButton: React.FC<AddToCartButtonProps> = (props) => {
	const { id } = props;
	{ /* зависимость от какого-то сервиса добавления в корзину */ }
	const { addToCart, process } = useFetchAddToCart();
	{ /* зависимость от какого-то глобального стейта корзины */ }
	const inCartAmount = useAmountProductInCart(id);
		

	return (
		<Button 
			onClick={() => addToCart(id)} 
			loading={process}
		>
			{ inCartAmount ? `В корзине ${ inCartAmount }` : `Добавить в корзину` }
		</Button>
	);
};
```
И тут мы теряем всё. Зависим от большого количества того, что никак к карточке не относится.

Не можем легко сделать новое поведение при клике на карточку. (да, тут можем быстро переделать, но давайте представим, что не можем)).

Зависим от AddToCartButton, которая, скорее всего, зависит от какого-то глобального состояния Cart, а так же, возможно, например, от каких то глобальных сервисов для добавления товаров в корзину.

Почему это плохо?

- Это сложнее тестировать. Нужно всё оборачивать в мок-стейты, в мок-сервисы.. Но это как будто то что не должно так быть.

- Сложнее менять

- Допустим мы захотели в панели редактирования товаров показывать превью будущего товара в виде карточки, то как это будет отображаться в списках карточек итд. А в той части приложения у нас нету этого глобального стейта Cart и сервисов для добавления в корзину. И всё. Смэрть. А это всего лишь карточка товара и всего лишь один AddToCart. А еще может быть добавление в избранное.. Сравнение товаров.. итд..

## 2. Контейнеры (Containers)

Containers - это грязные компоненты. Они объявляю в себе что угодно и зависят от чего угодно. Служат обертками над чистыми компонентами. В приложении их должно быть минимально возможное количество.

Например:

```typescript jsx
export type ProductListContainerProps = {
    type: string;
    limit: number;
    page: number;
};

export const ProductListContainer: React.FC<ProductViewContainer> = (props) => {
    const { id } = props;
    { /* зависимость от сервиса поиска продуктов */ }
    const { loading, items }: FetchList<Product> = useFetchProductsByType({ type, limit, page });
    { /* зависимость от глобального стейта корзины */ }
    const { addToCartProduct }: ICartService = useCart();
    { /* зависимость от глобальной модалки */ }
    const { productCardClickHandler }: IProductPreview = useProductPreviewModal();

    if (loading) {
        return <Loader/>
    }

    if (products.length) {
        return (
            <ProductsCardList>
                {
                    products.map((product) => (
                        <ProductCard
                            key={product.id}
                            product={product}
                            onClick={productCardClickHandler}
                            header={<ImageSlider images={product.images}/>}
                            footer={<AddToCartButton id={product.id} onClick={addToCartProduct}/>}
                        />
                    ))
                }
            </ProductsCardList>
        );
    }

    return <NoResults/>;
};
```


Я не уверен, что их стоит тестировать. А защитить их можно интерфейсами и типами.

## 3. Макеты (Layouts)

Layouts - могут быть грязными компонентами или чистыми.

В макетах объявляется лишь то что нужно для самого макета, но не более того.

Например:
```typescript jsx
export const NavigationLayout = () => {
	const { pathname } = useLocation();

	return (
		<div className={css.container}>
			{
				pathname !== '/cart' &&
				<NavigationPannel/>
			}
			<div className={css.content}><Outlet/></div>
		</div>
	)
};
```

Я не уверен, что их стоит тестировать. А защитить их можно интерфейсами и типами.

## 4. Страницы (Pages)

Pages - могут быть грязными компонентами или чистыми, но на практике чистыми никогда не будут)

В страницах объявляется лишь то что относится к параметрам страницы, но не более того.

Например:
```typescript jsx
export const ProductsListPage = () => {
	const { type } = useParams<{ type: string }>();
	const { page, limit } = useProductListSearchParams();
	// ...

	return (
		<ProductsListContainer type={type} page={page} limit={limit}/>
	)
};
```

Я не уверен, что их стоит тестировать. А защитить их можно интерфейсами и типами.

Вложенность компонентов должна быть примерно такой:

Layouts → Pages → Containers → Components

Почему примерно? В Pages могут быть дальше Layouts, а в Layouts напрямую Containers.Главное правило, что в Components лежат ТОЛЬКО чистые компоненты.

Вот и всё.

Мой итог таков: разрабатывая по, по сути, единственному главному принципу, что максимальное количество компонентов должно быть чистыми и что чистые компоненты содержат в себе только чистые компоненты и хуки (функции) - мы получаем гораздо более легкую, поддерживаемую, гибкую, тестируемую, переиспользуемую систему. Ну и разделение на layouts, pages и containers, добавляет порядка. Да, у этого есть и свои недостатки.

- Больше кода писать

- На кажущуюся простоту, писать так сложнее (ну по крайней мере в начале). Особенно учитывая, что примеры игрушечные и прям очень простые.

Но всё это ничто по сравнению с теми проблемами с которыми вы можете столкнуться в будущем. И да, для маленьких проектов, возможно, это чрезмерно, но для чего-то большого, как будто это необходимо.

Как я уже писал - я сам только недавно до этого дошел и не уверен, что это лучший (или в принципе правильный) вариант. Так что хотелось бы узнать ваше мнение.

Так же. Считайте весь код - псевдокодом. Он был написан в блокноте в 3 часа ночи и там может быть много ошибок. о.о надо было, наверное, это в начале написать..

*************

Дальше о том, почему я вообще это всё написал.

До недавнего времени я учился только (90%) по урокам из YouTube. Как я сейчас уже понял, многие уроки там — не учат тебя делать правильно. Из‑за этого, когда ты садишься делать какой‑то более менее большой проект — возникает КУЧА проблем из‑за того, что ты просто делаешь так, как показывали в этих уроках, а это не правильно. И я очень долгое время не понимал этого и из‑за чего многие проекты (в учебных целях), которые я начинал, умирали.. Потому что разрабатывать становилось сложно.

И надеюсь, что этот материал поможет кому-нибудь.

Спасибо за внимание :-)

