# Blazor Puzzle #42

## Impossible CSS

YouTube Video: https://youtu.be/

Blazor Puzzle Home Page: https://blazorpuzzle.com

### The Challenge:

This is a .NET 8 Blazor Web App with Global Server Interactivity.

We have a `<div>` with a scrollbar to show more items than fit on a page.

We want to be able to keep the size of the `<div>` consistent when the browser is resized.

We have set the height to 80% of the view height (`height:80vh;`), but it doesn't quite work.

How can we make  this work?

### The Solution:

There were a few ways to come at this problem, but this is the one Carl chose.

*Home.razor*:

```c#
@page "/"
@inject  IJSRuntime jsRuntime

<HeadContent>
	<style>
		.myDiv {
			padding: 10px;
			overflow-y: auto;
			width: 400px;
			background-color: antiquewhite;
		}
	</style>
	<script>
		window.InitBlazorPage = (dotnetHelper) => {
            window.addEventListener('resize', () => {
                dotnetHelper.invokeMethodAsync('OnResize', window.innerWidth, window.innerHeight);
            });
            window.dispatchEvent(new Event('resize'));
        };

	</script>
</HeadContent>

<PageTitle>Home</PageTitle>

<p>
	This is a .NET 8 Blazor Web App with global server interactivity.
</p>
<p>
	How can we make this div consitently span the full height of the page, even if the browser is resized?
</p>


<div class="myDiv" style="height:@height">
	@for (int i = 0; i < 100; i++)
	{
		<p>Line @i</p>
	}
</div>

@code 
{
	string height = "";

	[JSInvokable]
	public async Task OnResize(int width, int height)
	{
		this.height = (height - 180).ToString() + "px";
        await InvokeAsync(StateHasChanged);
	}

	protected override async Task OnAfterRenderAsync(bool firstRender)
	{
		if (firstRender)
		{
			await jsRuntime.InvokeVoidAsync("InitBlazorPage", DotNetObjectReference.Create(this));
		}
	}
}
```

The `height` property was removed from the `myDiv` class:

```html
<style>
    .myDiv {
        padding: 10px;
        overflow-y: auto;
        width: 400px;
        background-color: antiquewhite;
    }
</style>
```

Instead, we defined a local `height` string variable:

```c#
string height = "";
```

The div itself uses the `height` variable in the style:

```html
<div class="myDiv" style="height:@height">
	@for (int i = 0; i < 100; i++)
	{
		<p>Line @i</p>
	}
</div>
```

To calculate `height`, we use JavaScript to hook the `resize` event:

```html
<script>
    window.InitBlazorPage = (dotnetHelper) => {
        window.addEventListener('resize', () => {
            dotnetHelper.invokeMethodAsync('OnResize', window.innerWidth, window.innerHeight);
        });
        window.dispatchEvent(new Event('resize'));
    };

</script>
```

We call the   `InitBlazorPage` JavaScript method from the page's `OnAfterRenderAsync` overload:

```c#
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await jsRuntime.InvokeVoidAsync("InitBlazorPage", DotNetObjectReference.Create(this));
    }
}
```

The JavaScript calls our `OnResize` method when the browser is resized, passing the width and height:

```c#
[JSInvokable]
public async Task OnResize(int width, int height)
{
    this.height = (height - 180).ToString() + "px";
    await InvokeAsync(StateHasChanged);
}
```

The result is that the `<div>` height is automatically set whenever the browser window is resized.

Boom!