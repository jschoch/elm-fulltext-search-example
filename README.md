#  example of full text search in elm 0.17

* [demo available here](https://s3-us-west-2.amazonaws.com/elmschoch/index.html)

```elm

 {-| Create an index and add multiple documents.
Copyright (c) 2016 Robin Luiten
-}

import ElmTextSearch
import Html exposing (Html, div, br, text,button, input, li, ul)
import Html.App as Html
import Html.App as App
import Html.Events exposing (onInput, onClick)
import Html.Attributes exposing (..)
import Dict


{-| Example document type. -}
type alias ExampleDocType =
  { cid : String
  , title : String
  , author : String
  , body : String
  }


{-| Create an index with default configuration.
See ElmTextSearch.SimpleConfig documentation for parameter information.
-}
createNewIndexExample : ElmTextSearch.Index ExampleDocType
createNewIndexExample =
  ElmTextSearch.new
    { ref = .cid
    , fields =
        [ ( .title, 5.0 )
        , ( .author, 1.0)
        , ( .body, 1.0 )
        ]
    }

init_dict = Dict.fromList [
        (id1.cid,id1)
        ,(id2.cid ,id2)
        ,(id3.cid,id3)
  ]

id1 : ExampleDocType
id1 = { cid = "id1"
    , title = "First Title Joe"
    , author = "Some Author"
    , body = "Words in this example document with explanations."
    }

id2 : ExampleDocType
id2 = { cid = "id2"
    , title = "Is a cactus as pretty as a tree ?"
    , author = "Joe Greeny"
    , body = "This title contains information about cactuses."
    }

id3 : ExampleDocType
id3 = { cid = "id3"
    , title = "bork bork Joe"
    , author = "Joe Greeny"
    , body = "foo bar baz."

  }

documents = [id1,id2,id3]

type alias Model = {search : String,results : String,new_dict : Dict.Dict String ExampleDocType }

type Msg = Search String
 | Go

update : Msg -> Model -> (Model, Cmd Msg)
update msg model = 
  case msg of
    Search s ->
      ({model | search = s},Cmd.none)
    Go -> 
      (model,Cmd.none)
{-| Add a documents to index.
If any add result is an Err this returns the first failure.
-}
indexWithMulitpleDocumentsAdded : (ElmTextSearch.Index ExampleDocType, List (Int, String))
indexWithMulitpleDocumentsAdded =
  ElmTextSearch.addDocs
    documents
    createNewIndexExample


{-| Search the index.
The result includes an updated Index because a search causes internal
caches to be updated to improve overall performance.
This is ignoring any errors from call to addAllDocs
in indexWithMulitpleDocumentsAdded.
-}
resultSearchIndex :
  String -> 
  Result String
    ( ElmTextSearch.Index ExampleDocType
    , List (String, Float)
    )
resultSearchIndex s =
  ElmTextSearch.search s (fst indexWithMulitpleDocumentsAdded)


{-| Display search result. -}
main =
  App.program {
    init = ({search = "title", results = "", new_dict = init_dict},Cmd.none)
    , view = view
    , update = update
    , subscriptions = sub
 }

sub : Model -> Sub Msg
sub model = Sub.none

view : Model -> Html Msg
view model = 
  let
    -- want only the search results not the returned index

    --search = model.search
    --case search of
        --"" -> keys = Dict.keys model.new_dict
        --_ -> 
    searchResults = Result.map snd (resultSearchIndex model.search) 
      |> Result.toMaybe
      |> Maybe.withDefault []
    resultsDict = Dict.fromList searchResults
    result_keys = Dict.keys resultsDict
    keys = if model.search == "" then
      Dict.keys model.new_dict
    else result_keys
  in
    div []
    [ text "Search"
      , input [ type' "text" , onInput Search, value model.search ] [ ]
      , br [] []
      ,text
      ( "Result of searching for \"explanations\" is "
          ++ (toString keys)
      )
      , ul [] (List.map (\x -> li [] [ text (toString (Dict.get x model.new_dict))]) keys)
    ]

filteredList : Model -> String
filteredList model = 
        let
          f = List.filter (\x -> view_doc x model.new_dict) (Dict.keys model.new_dict)
        in
          toString f


view_doc : String -> Dict.Dict String ExampleDocType -> Bool
view_doc id dict =
       case Dict.get id dict of
             Just doc -> True
             Nothing -> False
```
