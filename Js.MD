## Jest testing
#### Импортируем библиотеку
````
import {render, screen, fireEvent, waitFor, cleanup} from "@testing-library/react";
````
#### Создаем моки
````
global.fetch = jest.fn();
jest.mock("../assets/img/trash.png", () => "trash")
jest.mock("../helps/logs.mjs", () => jest.fn())
jest.mock("../components/admin/AdminOrderInvoiceModal", () => () => <div data-testid="modal-invoice" />);
const alertMock = jest.spyOn(window, "alert").mockImplementation(() => {
});
delete window.location;
window.location = {reload: jest.fn()};
````
#### Создаем для теста
````
describe("Dample component", () => {
  const mockSample = [
    {
      id: 1,
      name: "Product A",
      description: "Desc A",
      amount: 10,
      service: 1,
      price: 100
    }];

  beforeEach(() => {
    fetch.mockResolvedValueOnce({
      json: () => Promise.resolve({
        success: true,
        data: { products: mockProducts }
      })
    });

    render(<AdminProducts />);
  });

  afterEach(() => {
    jest.clearAllMocks();
    cleanup();
  });

  
  it("calls fetchUpdate when clicking Update", async () => {
    fetch.mockResolvedValueOnce({
      json: () => Promise.resolve({ success: true })
    });

    const updateButtons = await screen.findAllByTestId("item_update");
    fireEvent.click(updateButtons[0]);

    await waitFor(() => {
      expect(fetch).toHaveBeenCalledWith(
        expect.stringContaining("/product/update/1"),
        expect.objectContaining({
          method: "POST",
          body: JSON.stringify({
            name: "Product A",
            description: "Desc A",
            amount: 10,
            service: "1",
            price: 100
          })
        })
      );
    });
  });

  it("updates input value on change", async () => {
    const input = await screen.findByDisplayValue("Product B");
    fireEvent.change(input, { target: { value: "Product B Updated" } });
    expect(input.value).toBe("Product B Updated");
  });
});

````

