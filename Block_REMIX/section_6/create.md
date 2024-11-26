# Creating items in the library database

In this section the routes required to create new items in the database will be implemented.

## Adding links to the root page

The first step is to add links to the root page to the create pages.  This can be done by adding an array named create to the root.tsx file.

**root.tsx (extract)**
```javascript
  const create = [
    ['/catalog/books/create', 'Create book'],
    ['/catalog/authors/create', 'Create author'],
    ['/catalog/genres/create', 'Create genre'],
    ['/catalog/instances/create', 'Create book-instance']
  ]
```
A new card is then added to the first column of the home page to display the links to the create pages.

**root.tsx (extract)**
```javascript
              <h2>Create</h2>
              <Card className="text-left" border="primary">
                <Card.Body>
                  <ul>
                    <nav>
                      {create.map((retriever) => (
                        <li key={retriever[1]}>
                          <NavLink
                            className={({ isActive, isPending }) =>
                              isActive
                                ? "active" : isPending
                                  ? "pending" : ""
                            }
                            to={retriever[0]}
                          >
                            {retriever[1]}
                          </NavLink>
                        </li>
                      ))}
                    </nav>
                  </ul>
                </Card.Body>
              </Card>
            </ Col>
```

The full listing for the root.tsx file is shown below.
**root.tsx (full listing)**
```javascript
//import {redirect } from "@remix-run/node";
import { Container, Row, Col } from 'react-bootstrap';
import Card from 'react-bootstrap/Card';

import {
  Links,
  Meta,
  NavLink,
  Outlet,
  Scripts,
  ScrollRestoration,
} from "@remix-run/react";

import type { MetaFunction } from "@remix-run/node";
// existing imports

import "./app.css"; // includes bootstrap

export const meta: MetaFunction = () => {
  return [
    { title: "Local Library" },
    {
      property: "og:title",
      content: "Very cool app",
    },
    {
      name: "description",
      content: "Acess local library",
    },
  ];
};


export default function App() {

  const retrieve = [
    ['/', 'home'],
    ['/catalog/', 'Catalog'],
    ['/catalog/books/', 'All books'],
    ['/catalog/authors/', 'All authors'],
    ['/catalog/genres/', 'All genres'],
    ['/catalog/instances/', 'All book-instances']

  ]

  const create = [
    ['/catalog/books/create', 'Create book'],
    ['/catalog/authors/create', 'Create author'],
    ['/catalog/genres/create', 'Create genre'],
    ['/catalog/instances/create', 'Create book-instance']
  ]

  return (
    <html lang="en">
      <head>
        <Meta />
        <Links />
      </head>
      <body>
        {/* comments in JSX are in braces  */}
        <Container fluid="md">
          <Row >
            <Col xs={3} >
              <h1>Local Library</h1>
              <Card className="text-left" border="primary">
                <Card.Body>
                  <ul>
                    <nav>
                      {retrieve.map((retriever) => (
                        <li key={retriever[1]}>
                          <NavLink
                            className={({ isActive, isPending }) =>
                              isActive
                                ? "active" : isPending
                                  ? "pending" : ""
                            }
                            to={retriever[0]}
                          >
                            {retriever[1]}
                          </NavLink>
                        </li>
                      ))}
                    </nav>
                  </ul>
                </Card.Body>
              </Card>

              <h2>Create</h2>
              <Card className="text-left" border="primary">
                <Card.Body>
                  <ul>
                    <nav>
                      {create.map((retriever) => (
                        <li key={retriever[1]}>
                          <NavLink
                            className={({ isActive, isPending }) =>
                              isActive
                                ? "active" : isPending
                                  ? "pending" : ""
                            }
                            to={retriever[0]}
                          >
                            {retriever[1]}
                          </NavLink>
                        </li>
                      ))}
                    </nav>
                  </ul>
                </Card.Body>
              </Card>
            </ Col>

            <Col >
              <Outlet />
              <ScrollRestoration />
              <Scripts />
            </Col>
          </Row>
        </Container>
      </body>
    </html>
  );
}
```

## Create a genre

The genre only has a single fild, the name of the genre, so this is the simplest to create as a first step.

To create a genre, a form is required with space for the genre name to be entered.
The form should be validated so that the data entered is valid (even if not correct!). The form should be validated on the client and on the server.

The [remix-validated-form](https://www.rvf-js.io/remix) library is used to validate the form.  This allows the form to be validated on the client and on the server.

The validation can be done using a validation adaptor for [zod](https://zod.dev/) or [yup](https://github.com/jquense/yup).  I will use zod.

The link to ```/catalog/genres/create``` will correspond to a route handler that will be created in the ```routes/catalog/genres_.create.tsx``` file.

This file will start by importing the required libraries and the validation schema.

**routes/catalog/genres_.create.tsx (extract)**
```javascript
import { Container } from 'react-bootstrap';
import { withZod } from "@rvf/zod";
import { z } from "zod";
import { useForm, validationError, } from "@rvf/remix";
import Genre from '../models/genre';
import type { ActionFunctionArgs, } from "@remix-run/node";
import { redirect,json } from "@remix-run/node";
import { FormInput, FormSubmit } from '../components/formUI';
```
The validation schema is defined using zod.  In this case the genre name is required and must be a string of at least 1 character and up to 20 characters.  The trim() will remove any leading or trailing spaces.

This is a simple example of a validation schema.  In a real application, more detailed validation would be required.

**routes/catalog/genres_.create.tsx (extract)**
```javascript
export const validator = withZod(
    z.object({
        name: z.string().min(1, { message: "Genre name is required" }).max(20).trim()
    })
);
```
When the route is called the first action is to wait for the form data to be submitted. The data from the form is read into a constant called newGenre. 

If there is no validation error the database is checked to see if a genre with the same name already exists.  If a genre with the same name already exists, an error is returned.  If a genre with the same name does not exist, a new genre is created using the newGenre.save() method. 
A function createGenre is defined which will take form data as a parameter to create a new genre.  

**routes/catalog/genres_.create.tsx (extract)**
```javascript
async function createGenre(formData: FormData) {
    const newGenre = new Genre({
        name: formData.get('name')?.toString()
    });
    const instances = await Genre.find({ name: formData.get('name')?.toString() }, null, { limit: 1 }).exec();

    if (instances.length > 0) {
        console.log(instances);
        return {
            status: 400,
            body: 'Genre already exists'
        }
    } else {
        await newGenre.save();
    }
    return;
}
```

The file then exports a function called action.  This function takes the request data from a submitted form and 
validates it using the validator.  If there is no validation error the createGenre function is called with the form data.  If there is a validation error the result will be returned and the error message set in the validatior will be displayed on the form.

Finally the file provides a description of the form as it will be displayed on the page by the function GenresForm().  This provides the jsx code for the form.

**routes/catalog/genres_.create.tsx (extract)**
```javascript
const GenresForm = () => {
    const form = useForm({
        validator,
        method: "post"
    });

    return (
        <form {...form.getFormProps()} >
            <h1>Enter new genre details</h1>
            <Container>
                <FormInput label="Genre Name" name="name" form={form} />   
                <FormSubmit form={form} />
            </Container>
        </form>
    );
}
```

All the above is private to this module.  The defaulot function exports a function which will place the form on the page on the page within <GenresForm />.  

The actions of the form are then run automatically when the form is submitted.
**routes/catalog/genres_.create.tsx (extract)**
```javascript
export default function NewGenre() {
    return (
        <GenresForm />
    );
}
```
The full listing of the file ```routes/catalog/genres_.create.tsx``` is shown below.

**routes/catalog/genres_.create.tsx (full listing)**
```javascript
import { Container } from 'react-bootstrap';
import { withZod } from "@rvf/zod";
import { z } from "zod";
import { useForm, validationError, } from "@rvf/remix";
import Genre from '../models/genre';
import type { ActionFunctionArgs, } from "@remix-run/node";
import { redirect,json } from "@remix-run/node";
import { FormInput, FormSubmit } from '../components/formUI';


export const validator = withZod(
    z.object({
        name: z.string().min(1, { message: "Genre name is required" }).max(20).trim()
    })
);

async function createGenre(formData: FormData) {
    const newGenre = new Genre({
        name: formData.get('name')?.toString()
    });
    const instances = await Genre.find({ name: formData.get('name')?.toString() }, null, { limit: 1 }).exec();

    if (instances.length > 0) {
        console.log(instances);
        return {
            status: 400,
            body: 'Genre already exists'
        }
    } else {
        await newGenre.save();
    }
    return;
}


export const action = async ({
    request,
}: ActionFunctionArgs) => {
    const formData = await request.formData();
    const result = await validator.validate(formData);

    if (result.error) return validationError(result.error, result.submittedData);

    const duplicate = await createGenre(formData);
    console.log('duplicate');
    console.log(duplicate);
    if (duplicate) {
        return json({ error: duplicate.body });
    } else if (duplicate === undefined) {
        return redirect('/catalog/genres');
    }
}


const GenresForm = () => {
    const form = useForm({
        validator,
        method: "post"
    });

    return (
        <form {...form.getFormProps()} >
            <h1>Enter new genre details</h1>
            <Container>
                <FormInput label="Genre Name" name="name" form={form} />   
                <FormSubmit form={form} />
            </Container>
        </form>
    );
}


export default function NewGenre() {
    return (
        <GenresForm />
    );
}
```

In the same way the full listings of the files for all the other 'create' routes are shown below and you should study these to follow the logic of the code.

## Authors

The full listing of the file ```routes/catalog/authors_.create.tsx``` is shown below.
```javascript
// https://remix.run/docs/en/main/guides/data-writes
// https://www.remix-validated-form.io/integrate-your-components
//https://remix.run/docs/en/main/guides/data-writes
//https://www.rvf-js.io/input-types


import { Container } from 'react-bootstrap';
import { withZod } from "@rvf/zod";
import { z } from "zod";
import { useForm, validationError, } from "@rvf/remix";

import Author from '../models/author';
import type { ActionFunctionArgs, } from "@remix-run/node";
import { redirect, } from "@remix-run/node";
import { FormInput, FormSubmit } from '../components/formUI';

const validator = withZod(
    z.object({
        first_name: z.string().min(1, { message: "First name is required" }).max(100).trim(),
        family_name: z.string().min(1, { message: "Last name is required" }).max(100).trim(),
        date_of_birth: z.coerce.date({
            required_error: "Date is required.",
            invalid_type_error: "Wrong date format.",
        }),
        date_of_death: z.union([
            z.string().length(0, { message: "Enter numeric date" }).nullable(),
            z.coerce.date({
                invalid_type_error: "Wrong date format.",
            })
        ]).or(z.undefined()).optional(),
    })
);

async function createAuthor(formData: FormData) {
    const newAuthor = new Author({
        first_name: formData.get('first_name'),
        family_name: formData.get('family_name'),
        date_of_birth: formData.get('date_of_birth'),
        date_of_death: formData.get('date_of_death'),
    });
    await newAuthor.save();
    return;
}


export const action = async ({
    request,
}: ActionFunctionArgs) => {
    const formData = await request.formData();
    const data = await validator.validate(formData);
    console.log(data.error);
    console.log(data.submittedData);
    if (data.error) return validationError(data.error, data.submittedData);
    await createAuthor(formData);
    return redirect('/catalog/authors');
};


const AuthorsForm = () => {
    const form = useForm({
        validator,
        method: "post"
    });

    return (
        <form {...form.getFormProps()} >
            <h1>Enter new author details</h1>
            <Container>
                <FormInput label="First Name" name="first_name" form={form} />
                <FormInput label="Last Name" name="family_name" form={form} />
                <FormInput label="Date of Birth" name="date_of_birth" form={form} />
                <FormInput label="Date of Death" name="date_of_death" form={form} />    
                <FormSubmit form={form} />
            </Container>
        </form>
    );
}


export default function NewAuthor() {
    return (
        <AuthorsForm />
    );
}
```

## Books
The full listing of the file ```routes/catalog/books_.create.tsx``` is shown below.

**routes/catalog/books_.create.tsx**
```javascript
//https://www.remix-validated-form.io/repeated-field-names
//https://www.npmjs.com/package/zod-form-data/v/1.2.0

import { FormInput, SelectInput, FormSubmit } from '../components/formUI';
import { withZod } from "@rvf/zod";
import { z } from "zod";
import { zfd } from "zod-form-data";
import { useForm, validationError, } from "@rvf/remix";
import type { ActionFunctionArgs, } from "@remix-run/node";//
import { redirect, json } from "@remix-run/node";
import Author, { IAuthor } from '../models/author';
import Genre, { IGenre } from '../models/genre';
import Book from '../models/book';
import { Container } from 'react-bootstrap';
import { useLoaderData } from "@remix-run/react";


export const loader: unknown = async () => {

    const authors = await Author.find({}, null, { virtuals: true })
        .sort([['family_name', 'ascending']])
        .exec();

    if (!authors) {
        throw new Response("Not Found", { status: 404 });
    }

    const genres = await Genre.find({}, null, { virtuals: true })
        .sort([['name', 'ascending']])
        .exec();

    if (!genres) {
        throw new Response("Not Found", { status: 404 });
    }

    return json({ authors, genres });
};

export const validator = withZod(
    z.object({
        title: z.string().min(1, { message: "Title is required" }).max(40).trim(),
        authors: zfd.repeatable(z.array(zfd.text()).min(1, { message: "Author selection is required" })),
        summary: z.string().min(1, { message: "Summary is required" }).max(1000).trim(),
        isbn: z.string().min(1, { message: "ISBN is required" }).max(13).trim(),
        genres: zfd.repeatable(z.array(zfd.text()).min(1, { message: "Genre selection is required" }))
    })
);


async function createBook(formData: FormData) {
    const newBook = new Book({
        title: formData.get('title'),
        authors: formData.getAll('authors'),
        summary: formData.get('summary'),
        isbn: formData.get('isbn'),
        genres: formData.getAll('genres'),
    });
    await newBook.save();
    return;
}


export const action = async ({
    request,
}: ActionFunctionArgs) => {
    const formData = await request.formData();
    console.log("submitted book details")
    console.log(formData.get('title'));
    console.log(formData.getAll('authors'));
    console.log(formData.get('summary'));
    console.log(formData.get('isbn'));
    console.log(formData.getAll('genres'));
    const result = await validator.validate(formData);

    if (result.error) return validationError(result.error, result.submittedData);

    await createBook(formData);
    return redirect('/catalog/books');

};

const BooksForm = () => {
    const data = useLoaderData() as { authors: IAuthor[], genres: IGenre[] };
    const form = useForm({ validator, method: "post" });
    return (
        <form {...form.getFormProps()} >

            <h1>Enter new book details</h1>
            <Container>
                <FormInput   name="title"   label="Book Title" form={form} />
                <SelectInput name="authors" label="Authors"    form={form} data={data.authors} /> 
                <FormInput   name="summary" label="Summary"    form={form} />
                <FormInput   name="isbn"    label="ISBN"       form={form} />
                <SelectInput name="genres"  label="Genres"     form={form} data={data.genres} />
                <FormSubmit form={form} />
            </Container>
        </form>
    );
}

export default function NewBook() {
    return (
        <BooksForm />
    );
}
```

## BookInstance

It is left to you to create the ```routes/catalog/bookinstances_.create.tsx``` file.








